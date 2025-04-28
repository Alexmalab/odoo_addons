# Odoo Module: website_payment

Category: Hidden

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

{
    'name': 'Website Payment',
    'category': 'Hidden',
    'summary': 'Payment integration with website',
    'version': '1.0',
    'description': """
This is a bridge module that adds multi-website support for payment providers.
    """,
    'depends': [
        'website',
        'account_payment',
        'portal',
    ],
    'data': [
        'data/mail_templates.xml',
        'data/mail_template_data.xml',

        'views/payment_form_templates.xml',
        'views/payment_provider.xml',
        'views/res_config_settings_views.xml',
        'views/snippets/snippets.xml',
        'views/snippets/s_donation.xml',
    ],
    'auto_install': True,
    'assets': {
        'website.assets_wysiwyg': [
            'website_payment/static/src/snippets/s_donation/options.js',
            'website_payment/static/src/snippets/s_donation/options.xml',
        ],
        'web.assets_frontend': [
            'website_payment/static/src/js/**/*',
        ],
        'web.assets_tests': [
            'website_payment/static/tests/tours/donation.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http, _
from odoo.exceptions import ValidationError
from odoo.http import request
from odoo.tools.json import scriptsafe as json_safe

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment.controllers import portal as payment_portal


class PaymentPortal(payment_portal.PaymentPortal):
    @http.route('/donation/pay', type='http', methods=['GET', 'POST'], auth='public', website=True, sitemap=False)
    def donation_pay(self, **kwargs):
        """ Behaves like PaymentPortal.payment_pay but for donation

        :param dict kwargs: As the parameters of in payment_pay, with the additional:
            - str donation_options: The options settled in the donation snippet
            - str donation_descriptions: The descriptions for all prefilled amounts
        :return: The rendered donation form
        :rtype: str
        :raise: werkzeug.exceptions.NotFound if the access token is invalid
        """
        kwargs['is_donation'] = True
        kwargs['currency_id'] = self._cast_as_int(kwargs.get('currency_id')) or request.env.company.currency_id.id
        kwargs['amount'] = self._cast_as_float(kwargs.get('amount')) or 25.0
        kwargs['donation_options'] = kwargs.get('donation_options', json_safe.dumps(dict(customAmount="freeAmount")))

        if request.env.user._is_public():
            kwargs['partner_id'] = request.env.user.partner_id.id
            kwargs['access_token'] = payment_utils.generate_access_token(kwargs['partner_id'], kwargs['amount'], kwargs['currency_id'])

        return self.payment_pay(**kwargs)

    @http.route('/donation/transaction/<minimum_amount>', type='json', auth='public', website=True, sitemap=False)
    def donation_transaction(self, amount, currency_id, partner_id, access_token, minimum_amount=0, **kwargs):
        if float(amount) < float(minimum_amount):
            raise ValidationError(_('Donation amount must be at least %.2f.', float(minimum_amount)))
        use_public_partner = request.env.user._is_public() or not partner_id
        if use_public_partner:
            details = kwargs['partner_details']
            if not details.get('name'):
                raise ValidationError(_('Name is required.'))
            if not details.get('email'):
                raise ValidationError(_('Email is required.'))
            if not details.get('country_id'):
                raise ValidationError(_('Country is required.'))
            partner_id = request.website.user_id.partner_id.id
            del kwargs['partner_details']
        else:
            partner_id = request.env.user.partner_id.id

        self._validate_transaction_kwargs(kwargs, additional_allowed_keys=(
            'donation_comment', 'donation_recipient_email', 'partner_details', 'reference_prefix'
        ))
        if use_public_partner:
            kwargs['custom_create_values'] = {'tokenize': False}
        tx_sudo = self._create_transaction(
            amount=amount, currency_id=currency_id, partner_id=partner_id, **kwargs
        )
        tx_sudo.is_donation = True
        if use_public_partner:
            tx_sudo.update({
                'partner_name': details['name'],
                'partner_email': details['email'],
                'partner_country_id': int(details['country_id']),
            })
        elif not tx_sudo.partner_country_id:
            tx_sudo.partner_country_id = int(kwargs['partner_details']['country_id'])
        # the user can change the donation amount on the payment page,
        # therefor we need to recompute the access_token
        access_token = payment_utils.generate_access_token(
            tx_sudo.partner_id.id, tx_sudo.amount, tx_sudo.currency_id.id
        )
        self._update_landing_route(tx_sudo, access_token)

        # Send a notification to warn that a donation has been made
        recipient_email = kwargs['donation_recipient_email']
        comment = kwargs['donation_comment']
        tx_sudo._send_donation_email(True, comment, recipient_email)

        return tx_sudo._get_processing_values()

    def _get_extra_payment_form_values(
        self, donation_options=None, donation_descriptions=None, is_donation=False, **kwargs
    ):
        rendering_context = super()._get_extra_payment_form_values(
            donation_options=donation_options,
            donation_descriptions=donation_descriptions,
            is_donation=is_donation,
            **kwargs,
        )
        if is_donation:
            user_sudo = request.env.user
            logged_in = not user_sudo._is_public()
            # If the user is logged in, take their partner rather than the partner set in the params.
            # This is something that we want, since security rules are based on the partner, and created
            # tokens should not be assigned to the public user. This should have no impact on the
            # transaction itself besides making reconciliation possibly more difficult (e.g. The
            # transaction and invoice partners are different).
            partner_sudo = user_sudo.partner_id
            partner_details = {}
            if logged_in:
                partner_details = {
                    'name': partner_sudo.name,
                    'email': partner_sudo.email,
                    'country_id': partner_sudo.country_id.id,
                }

            countries = request.env['res.country'].sudo().search([])
            descriptions = request.httprequest.form.getlist('donation_descriptions')

            donation_options = json_safe.loads(donation_options) if donation_options else {}
            donation_amounts = json_safe.loads(donation_options.get('donationAmounts', '[]'))

            rendering_context.update({
                'is_donation': True,
                'partner': partner_sudo,
                'submit_button_label': _("Donate"),
                'transaction_route': '/donation/transaction/%s' % donation_options.get('minimumAmount', 0),
                'partner_details': partner_details,
                'error': {},
                'countries': countries,
                'donation_options': donation_options,
                'donation_amounts': donation_amounts,
                'donation_descriptions': descriptions,
            })
        return rendering_context

    def _get_payment_page_template_xmlid(self, **kwargs):
        if kwargs.get('is_donation'):
            return 'website_payment.donation_pay'
        return super()._get_payment_page_template_xmlid(**kwargs)

    @staticmethod
    def _compute_show_tokenize_input_mapping(providers_sudo, **kwargs):
        """ Override of `payment` to hide the "Save my payment details" input in the payment form
        when its a donation and user is not logged in.

        :param payment.provider providers_sudo: The providers for which to determine whether the
                                                tokenization input should be shown or not.
        :param dict kwargs: The optional data passed to the helper methods.
        :return: The mapping of the computed value for each provider id.
        :rtype: dict
        """
        res = super(PaymentPortal, PaymentPortal)._compute_show_tokenize_input_mapping(
            providers_sudo, **kwargs
        )
        if kwargs.get('is_donation') and request.env.user._is_public():
            for provider_sudo in providers_sudo:
                res[provider_sudo.id] = False
        return res

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">
    <template id="website_payment.donation_mail_body" name="Donation mail">
        <table border="0" cellpadding="0" style="background-color: white; padding: 0px; border-collapse:separate;">
            <tr style="height: 48px;"><td valign="top"><span style="font-size: 24px; font-weight: bold;">
                <t t-if="is_internal_notification">Donation notification</t>
                <t t-else="">Donation</t>
            </span></td></tr>
            <t t-if="not is_internal_notification">
                <tr><td valign="top">
                    Dear <t t-out="tx.partner_name"/>,
                </td></tr>
                <tr><td valign="top">
                    <div style="margin: 16px 0px 16px 0px;">
                        Thank you for your donation of <span t-out="tx.amount" t-options="{'widget': 'monetary', 'display_currency': tx.currency_id}"/> made on <t t-out="tx.create_date" t-options="{'widget': 'date'}"/>.
                        <br/>
                        We appreciate your support for our organization as such.
                        <br/>
                        Regards.
                    </div>
                </td></tr>
            </t>
            <tr><td valign="top">
                <div style="margin: 16px 0px 16px 0px;">
                    <table border="0" cellpadding="0" cellspacing="5" width="100%">
                        <tr>
                            <td><b>Donor Name:</b></td>
                            <td><t t-out="tx.partner_name"/></td>
                        </tr>
                        <tr>
                            <td><b>Donor Email:</b></td>
                            <td><t t-out="tx.partner_email"/></td>
                        </tr>
                        <tr>
                            <td><b>Donation Date:</b></td>
                            <td><t t-out="tx.create_date.date()"/></td>
                        </tr>
                        <tr>
                            <td><b>Amount(<t t-out="tx.currency_id.symbol"/>):</b></td>
                            <td><t t-out="tx.amount"/></td>
                        </tr>
                        <tr t-if="is_internal_notification and comment">
                            <td><b>Comment:</b></td>
                            <td><t t-out="comment"/></td>
                        </tr>
                        <tr>
                            <td><b>Payment Method:</b></td>
                            <td><t t-out="tx.provider_code"/></td>
                        </tr>
                        <tr>
                            <td><b>Payment ID:</b></td>
                            <td><t t-out="tx.reference"/></td>
                        </tr>
                    </table>
                </div>
            </td></tr>
        </table>
    </template>
</data>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mail_template_donation" model="mail.template">
            <field name="name">Website: Donation</field>
            <field name="model_id" ref="payment.model_payment_transaction" />
            <field name="lang">{{ object.partner_id.lang }}</field>
        </record>
    </data>
</odoo>

```

## File: models\account_payment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AccountPayment(models.Model):
    _inherit = 'account.payment'

    is_donation = fields.Boolean(string="Is Donation", related="payment_transaction_id.is_donation")

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.urls import iri_to_uri

from odoo import api, fields, models
from odoo.http import request

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment.const import REPORT_REASONS_MAPPING


class PaymentProvider(models.Model):
    _inherit = "payment.provider"

    website_id = fields.Many2one(
        "website",
        check_company=True,
        ondelete="restrict",
    )

    @api.model
    def _get_compatible_providers(self, *args, website_id=None, report=None, **kwargs):
        """ Override of `payment` to only return providers matching website-specific criteria.

        In addition to the base criteria, the website must either not be set or be the same as the
        one provided in the kwargs.

        :param int website_id: The provided website, as a `website` id.
        :param dict report: The availability report.
        :return: The compatible providers.
        :rtype: payment.provider
        """
        providers = super()._get_compatible_providers(
            *args, website_id=website_id, report=report, **kwargs
        )
        if website_id:
            unfiltered_providers = providers
            providers = providers.filtered(
                lambda p: not p.website_id or p.website_id.id == website_id
            )
            payment_utils.add_to_report(
                report,
                unfiltered_providers - providers,
                available=False,
                reason=REPORT_REASONS_MAPPING['incompatible_website'],
            )
        return providers

    def get_base_url(self):
        # Give priority to url_root to handle multi-website cases
        if request and request.httprequest.url_root:
            # Some domain names can use non-Latin script or alphabet or the Latin
            # alphabet-based characters with diacritics or ligatures. They are
            # stored as ASCII strings using Punycode transcription in the DNS
            # system and need to be converted to send to external APIs.
            return iri_to_uri(request.httprequest.url_root)
        return super().get_base_url()

    def copy(self, default=None):
        res = super().copy(default=default)
        if self._context.get('stripe_connect_onboarding'):
            res.website_id = False
        return res

```

## File: models\payment_transaction.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from markupsafe import Markup
from odoo import _, fields, models


class PaymentTransaction(models.Model):
    _inherit = "payment.transaction"

    is_donation = fields.Boolean(string="Is donation")

    def _post_process(self):
        super()._post_process()
        for donation_tx in self.filtered(lambda tx: tx.state == 'done' and tx.is_donation):
            donation_tx._send_donation_email()
            msg = [_('Payment received from donation with following details:')]
            for field in ['company_id', 'partner_id', 'partner_name', 'partner_country_id', 'partner_email']:
                field_name = donation_tx._fields[field].string
                value = donation_tx[field]
                if value:
                    if hasattr(value, 'name'):
                        value = value.name
                    msg.append(Markup('<br/>- %s: %s') % (field_name, value))
            donation_tx.payment_id._message_log(body=Markup().join(msg))

    def _send_donation_email(self, is_internal_notification=False, comment=None, recipient_email=None):
        self.ensure_one()
        if is_internal_notification or self.state == 'done':
            subject = _('A donation has been made on your website') if is_internal_notification else _('Donation confirmation')
            body = self.env['ir.qweb']._render('website_payment.donation_mail_body', {
                'is_internal_notification': is_internal_notification,
                'tx': self,
                'comment': comment,
            }, minimal_qcontext=True)
            self.env.ref('website_payment.mail_template_donation').send_mail(
                self.id,
                email_layout_xmlid="mail.mail_notification_light",
                email_values={
                    'email_to': recipient_email if is_internal_notification else self.partner_email,
                    'email_from': self.company_id.email_formatted,
                    'author_id': self.partner_id.id,
                    'subject': subject,
                    'body_html': body,
                },
                force_send=True,
            )

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    providers_state = fields.Selection(
        selection=[
            ('none', 'None'),
            ('paypal_only', 'Paypal Only'),
            ('other_than_paypal', 'Other than Paypal')
        ],
        compute='_compute_providers_state')
    first_provider_label = fields.Char(
        compute='_compute_providers_state')
    is_stripe_supported_country = fields.Boolean(
        related='company_id.country_id.is_stripe_supported_country')

    def _get_activated_providers(self):
        self.ensure_one()
        wire_transfer = self.env.ref('payment.payment_provider_transfer', raise_if_not_found=False)
        return self.env['payment.provider'].search([
            ('state', '!=', 'disabled'),
            ('id', '!=', wire_transfer.id if wire_transfer else False),
            '|',
            ('website_id', '=', False),
            ('website_id', '=', self.website_id.id)
        ])

    @api.depends('website_id')
    def _compute_providers_state(self):
        paypal = self.env.ref('payment.payment_provider_paypal', raise_if_not_found=False)
        stripe = self.env.ref('payment.payment_provider_stripe', raise_if_not_found=False)
        for config in self:
            providers = config._get_activated_providers()
            first_provider = stripe if stripe and stripe in providers else providers[0] if providers else providers
            config.first_provider_label = _('Configure %s', first_provider.name)
            if len(providers) == 1 and providers == paypal:
                config.providers_state = 'paypal_only'
            elif len(providers) >= 1:
                config.providers_state = 'other_than_paypal'
            else:
                config.providers_state = 'none'

    def action_activate_stripe(self):
        self.ensure_one()
        if not self.is_stripe_supported_country:
            return False
        menu = self.env.ref('website.menu_website_website_settings', raise_if_not_found=False)
        menu_id = menu and menu.id
        return self.env.company._run_payment_onboarding_step(menu_id=menu_id)

    def action_configure_first_provider(self):
        self.ensure_one()
        stripe = self.env['payment.provider'].search([
            *self.env['payment.provider']._check_company_domain(self.env.company),
            ('code', '=', 'stripe')
        ], limit=1)
        providers = self._get_activated_providers()
        return {
            'name': self.first_provider_label,
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'payment.provider',
            'views': [[False, 'form']],
            'res_id': stripe.id if stripe in providers else providers[0].id,
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_payment
from . import payment_provider
from . import payment_transaction
from . import res_config_settings

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><g clip-path="url(#o_icon_website_payment__a)"><path d="M26.001 34.5s-.83-2-.997-2.167c20.502-9.84 23.518-3.54 23.94-.955.068.418.005.843-.124 1.247A24.857 24.857 0 0 1 47 36.89c-1.5 1.5-10 1.61-10 1.61l-11-4Z" fill="#953B24"/><path d="m21 14 4.004 3.667c-20.502 9.84-23.518 3.54-23.94.955-.068-.418-.005-.843.124-1.247a24.84 24.84 0 0 1 1.428-3.514C6.713 5.644 14.501 11 14.501 11l6.5 3Z" fill="#1A6F66"/><path d="M1.187 32.625c-.129-.404-.192-.829-.124-1.247.422-2.586 3.438-8.886 23.94.955 14.197 6.815 20.01 5.89 22.382 3.818C43.286 44.36 34.803 50 25.003 50 13.855 50 4.411 42.703 1.187 32.625Z" fill="#FBB945"/><path d="M48.819 17.375c.129.404.192.829.124 1.247-.422 2.586-3.438 8.886-23.94-.955-14.197-6.815-20.01-5.89-22.382-3.818C6.72 5.64 15.203 0 25.003 0 36.15 0 45.594 7.297 48.819 17.375Z" fill="#1AD3BB"/></g><defs><clipPath id="o_icon_website_payment__a"><path fill="#fff" d="M0 0h50v50H0z"/></clipPath></defs></svg>

```

## File: static\shapes\s_donation_gift.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg"
  xmlns:xlink="http://www.w3.org/1999/xlink" id="e1a9485c-0f43-4303-9e0d-5e4a75181bc4" data-name="Layer 1" width="858.07" height="804.61" viewBox="0 0 858.07 804.61">
  <defs>
    <linearGradient id="fa2440bd-da04-40b7-bd3b-ddfb35b0a42b" x1="592.9" y1="385.11" x2="592.9" y2="236.35" gradientUnits="userSpaceOnUse">
      <stop offset="0" stop-color="gray" stop-opacity="0.25"/>
      <stop offset="0.54" stop-color="gray" stop-opacity="0.12"/>
      <stop offset="1" stop-color="gray" stop-opacity="0.1"/>
    </linearGradient>
    <linearGradient id="07fe1e9f-5680-428a-971a-ff84304916dd" x1="420.04" y1="804.61" x2="420.04" y2="320.21" xlink:href="#fa2440bd-da04-40b7-bd3b-ddfb35b0a42b"/>
    <linearGradient id="00ef2338-96af-4b11-80f8-5a4c3164adbb" x1="811.83" y1="471.54" x2="811.83" y2="185.81" xlink:href="#fa2440bd-da04-40b7-bd3b-ddfb35b0a42b"/>
  </defs>
  <title>gift1</title>
  <g opacity="0.5">
    <rect x="107.3" y="37.85" width="3.33" height="18.87" fill="#47e6b1"/>
    <rect x="276.52" y="77.53" width="3.33" height="18.87" transform="translate(195.93 -230.9) rotate(90)" fill="#47e6b1"/>
  </g>
  <g opacity="0.5">
    <rect x="811.66" y="769.53" width="3.33" height="18.87" fill="#47e6b1"/>
    <rect x="980.89" y="809.21" width="3.33" height="18.87" transform="translate(1631.97 -203.59) rotate(90)" fill="#47e6b1"/>
  </g>
  <g opacity="0.5">
    <rect x="812.98" y="215.34" width="3.33" height="18.87" fill="#47e6b1"/>
    <rect x="982.2" y="255.02" width="3.33" height="18.87" transform="translate(1079.1 -759.09) rotate(90)" fill="#47e6b1"/>
  </g>
  <g opacity="0.5">
    <rect x="846.98" y="45.34" width="3.33" height="18.87" fill="#47e6b1"/>
    <rect x="1016.2" y="85.02" width="3.33" height="18.87" transform="translate(943.1 -963.09) rotate(90)" fill="#47e6b1"/>
  </g>
  <g opacity="0.5">
    <rect x="267.08" y="36.27" width="3.33" height="18.87" fill="#47e6b1"/>
    <rect x="436.31" y="75.96" width="3.33" height="18.87" transform="translate(354.13 -392.27) rotate(90)" fill="#47e6b1"/>
  </g>
  <path d="M230.44,357.88a4.08,4.08,0,0,1-2.27-4.93,2,2,0,0,0,.09-.45h0a2,2,0,0,0-3.67-1.36h0a2,2,0,0,0-.23.4,4.08,4.08,0,0,1-4.93,2.27,2,2,0,0,0-.45-.09h0a2,2,0,0,0-1.36,3.67h0a2,2,0,0,0,.4.23,4.08,4.08,0,0,1,2.27,4.93,2,2,0,0,0-.09.45h0a2,2,0,0,0,3.67,1.36h0a2,2,0,0,0,.23-.4,4.08,4.08,0,0,1,4.93-2.27,2,2,0,0,0,.45.09h0a2,2,0,0,0,1.36-3.67h0A2,2,0,0,0,230.44,357.88Z" transform="translate(-169.23 -39.68)" fill="#4d8af0" opacity="0.5"/>
  <path d="M291.92,736.22a4.08,4.08,0,0,1-2.27-4.93,2,2,0,0,0,.09-.45h0a2,2,0,0,0-3.67-1.36h0a2,2,0,0,0-.23.4,4.08,4.08,0,0,1-4.93,2.27,2,2,0,0,0-.45-.09h0a2,2,0,0,0-1.36,3.67h0a2,2,0,0,0,.4.23,4.08,4.08,0,0,1,2.27,4.93,2,2,0,0,0-.09.45h0a2,2,0,0,0,3.67,1.36h0a2,2,0,0,0,.23-.4,4.08,4.08,0,0,1,4.93-2.27,2,2,0,0,0,.45.09h0a2,2,0,0,0,1.36-3.67h0A2,2,0,0,0,291.92,736.22Z" transform="translate(-169.23 -39.68)" fill="#4d8af0" opacity="0.5"/>
  <path d="M850.92,235.22a4.08,4.08,0,0,1-2.27-4.93,2,2,0,0,0,.09-.45h0a2,2,0,0,0-3.67-1.36h0a2,2,0,0,0-.23.4,4.08,4.08,0,0,1-4.93,2.27,2,2,0,0,0-.45-.09h0a2,2,0,0,0-1.36,3.67h0a2,2,0,0,0,.4.23,4.08,4.08,0,0,1,2.27,4.93,2,2,0,0,0-.09.45h0a2,2,0,0,0,3.67,1.36h0a2,2,0,0,0,.23-.4,4.08,4.08,0,0,1,4.93-2.27,2,2,0,0,0,.45.09h0a2,2,0,0,0,1.36-3.67h0A2,2,0,0,0,850.92,235.22Z" transform="translate(-169.23 -39.68)" fill="#4d8af0" opacity="0.5"/>
  <path d="M843.47,325.61a4.08,4.08,0,0,1-2.27-4.93,2,2,0,0,0,.09-.45h0a2,2,0,0,0-3.67-1.36h0a2,2,0,0,0-.23.4,4.08,4.08,0,0,1-4.93,2.27,2,2,0,0,0-.45-.09h0a2,2,0,0,0-1.36,3.67h0a2,2,0,0,0,.4.23,4.08,4.08,0,0,1,2.27,4.93,2,2,0,0,0-.09.45h0a2,2,0,0,0,3.67,1.36h0a2,2,0,0,0,.23-.4,4.08,4.08,0,0,1,4.93-2.27,2,2,0,0,0,.45.09h0a2,2,0,0,0,1.36-3.67h0A2,2,0,0,0,843.47,325.61Z" transform="translate(-169.23 -39.68)" fill="#4d8af0" opacity="0.5"/>
  <path d="M844,47.24a4.08,4.08,0,0,1-2.27-4.93,2,2,0,0,0,.09-.45h0a2,2,0,0,0-3.67-1.36h0a2,2,0,0,0-.23.4A4.08,4.08,0,0,1,833,43.18a2,2,0,0,0-.45-.09h0a2,2,0,0,0-1.36,3.67h0a2,2,0,0,0,.4.23,4.08,4.08,0,0,1,2.27,4.93,2,2,0,0,0-.09.45h0a2,2,0,0,0,3.67,1.36h0a2,2,0,0,0,.23-.4A4.08,4.08,0,0,1,842.56,51a2,2,0,0,0,.45.09h0a2,2,0,0,0,1.36-3.67h0A2,2,0,0,0,844,47.24Z" transform="translate(-169.23 -39.68)" fill="#4d8af0" opacity="0.5"/>
  <path d="M1012.16,504.28a4.08,4.08,0,0,1-2.27-4.93,2,2,0,0,0,.09-.45h0a2,2,0,0,0-3.67-1.36h0a2,2,0,0,0-.23.4,4.08,4.08,0,0,1-4.93,2.27,2,2,0,0,0-.45-.09h0a2,2,0,0,0-1.36,3.67h0a2,2,0,0,0,.4.23A4.08,4.08,0,0,1,1002,509a2,2,0,0,0-.09.45h0a2,2,0,0,0,3.67,1.36h0a2,2,0,0,0,.23-.4,4.08,4.08,0,0,1,4.93-2.27,2,2,0,0,0,.45.09h0a2,2,0,0,0,1.36-3.67h0A2,2,0,0,0,1012.16,504.28Z" transform="translate(-169.23 -39.68)" fill="#4d8af0" opacity="0.5"/>
  <circle cx="61.77" cy="166.12" r="6.66" fill="#f55f44" opacity="0.5"/>
  <circle cx="12.94" cy="644.43" r="6.66" fill="#f55f44" opacity="0.5"/>
  <circle cx="439.66" cy="45.68" r="6.66" fill="#f55f44" opacity="0.5"/>
  <circle cx="506.66" cy="172.68" r="6.66" fill="#f55f44" opacity="0.5"/>
  <circle cx="6.66" cy="478.68" r="6.66" fill="#f55f44" opacity="0.5"/>
  <circle cx="267.5" cy="155.34" r="6.66" fill="#47e6b1" opacity="0.5"/>
  <circle cx="725.6" cy="597.75" r="6.66" fill="#f55f44" opacity="0.5"/>
  <g opacity="0.5">
    <path d="M713.77,253.75c-12.9-12.94-31.71-18.09-53-14.5-19.89,3.36-39.79,14.13-56,30.33a65.49,65.49,0,0,0-11.72,16.08A65.64,65.64,0,0,0,581.43,269h0c-16-16.47-35.67-27.58-55.5-31.29-21.19-4-40.08.86-53.2,13.58S454.18,282.74,457.48,304C460.56,324,471,344,487,360.49,520.58,395.13,587.59,381.6,590.43,381l.84-.17,2.58.59c2.83.64,69.59,15.32,103.75-18.73,16.25-16.19,27.08-36.06,30.51-55.94C731.76,285.52,726.67,266.69,713.77,253.75ZM579.08,361c-16.19,2.18-56.55,5.14-76.59-15.53-12.88-13.29-21.31-29.19-23.71-44.75-2.2-14.2,1-26.27,8.94-34a29.93,29.93,0,0,1,10.92-6.69c6.68-2.37,14.63-2.81,23.32-1.18,15.48,2.89,31.1,11.81,44,25.1C586,304.67,581.76,344.92,579.08,361Zm127.78-57.93c-2.67,15.52-11.37,31.26-24.48,44.33-20.4,20.33-60.7,16.67-76.84,14.21-2.41-16.16-5.94-56.47,14.45-76.8,13.11-13.07,28.88-21.71,44.41-24.34,8.72-1.47,16.66-.9,23.3,1.58a29.93,29.93,0,0,1,10.8,6.88C706.33,276.82,709.3,288.94,706.86,303.09Z" transform="translate(-169.23 -39.68)" fill="url(#fa2440bd-da04-40b7-bd3b-ddfb35b0a42b)"/>
  </g>
  <path d="M596.91,376.65,590.53,378c-2.71.57-66.83,13.51-99-19.63-15.28-15.76-25.31-34.95-28.26-54-3.15-20.38,2-38.31,14.58-50.48s30.64-16.78,50.91-13c19,3.55,37.84,14.18,53.11,29.94h0c32.12,33.15,17.18,96.83,16.52,99.52ZM502.7,262.26a28.64,28.64,0,0,0-10.45,6.4c-7.62,7.38-10.66,18.93-8.56,32.52,2.3,14.89,10.36,30.1,22.69,42.82,19.17,19.78,57.79,17,73.28,14.86,2.57-15.41,6.61-53.93-12.56-73.71h0c-12.33-12.72-27.28-21.25-42.09-24C516.71,259.57,509.1,260,502.7,262.26Z" transform="translate(-169.23 -39.68)" fill="#3AADAA"/>
  <path d="M587.45,376.93,586,370.57c-.61-2.71-14.45-66.64,18.24-99.22h0c15.54-15.5,34.59-25.8,53.62-29,20.33-3.44,38.33,1.49,50.68,13.87s17.21,30.4,13.71,50.72c-3.28,19-13.65,38-29.19,53.53C660.39,393,596.51,379,593.8,378.38Zm96.17-112.75c-6.36-2.38-14-2.93-22.29-1.52-14.86,2.51-30,10.78-42.5,23.29h0c-19.51,19.45-16.14,58-13.83,73.49,15.45,2.35,54,5.86,73.53-13.6,12.55-12.51,20.87-27.57,23.43-42.42,2.34-13.55-.5-25.15-8-32.66A28.64,28.64,0,0,0,683.61,264.18Z" transform="translate(-169.23 -39.68)" fill="#3AADAA"/>
  <g opacity="0.5">
    <polygon points="730.87 320.21 109.21 320.21 109.21 444.6 141.51 444.6 141.51 804.61 698.57 804.61 698.57 444.6 730.87 444.6 730.87 320.21" fill="url(#07fe1e9f-5680-428a-971a-ff84304916dd)"/>
  </g>
  <rect x="148.56" y="357.82" width="542.96" height="440.67" fill="#f5f5f5"/>
  <rect x="368.89" y="436.51" width="102.3" height="361.98" fill="#3AADAA"/>
  <rect x="148.56" y="334.21" width="542.96" height="110.17" opacity="0.1"/>
  <rect x="117.08" y="326.34" width="605.91" height="110.17" fill="#f5f5f5"/>
  <rect x="368.89" y="326.34" width="102.3" height="110.17" fill="#3AADAA"/>
  <path d="M951.92,375.92C950,370.37,929,386,929,386l-4.84-.29-9.31-8.62,6.5-6.5-54.59-46.79a13.77,13.77,0,0,0-10.67-3.21l-45.37,5.67c-.19-2.6-1.8-5.69-4.06-8.84-4.65-7.63-14.45-16.26-18.08-19.33l-5.53-37.31,4.7-7.05a23.73,23.73,0,0,1-2.56-.86l1-1.49-.37-.1.47-.54-.06,0L798,237.16a5.14,5.14,0,0,0-.39-7.12l-26.21-19.88a14.45,14.45,0,0,0,1.12-6c.2-4-1.56-8-7.5-10.94a57.9,57.9,0,0,0-11.48-4.05,37.67,37.67,0,0,0-19.85.29l-.34-.4c-8.07-9.22-18.64,4-19.05,4.55l-41.34,46A8.4,8.4,0,0,0,675,250.81l21.88,16.41-.07.14,1.24.93c-.94,1.84-2.19,3.07-3.86,3.25l3.35,1.68a3.86,3.86,0,0,1-1.78.67l15.34,7.67c0,8,.82,44.47,13.79,62.39A33.13,33.13,0,0,0,729.6,350s1.1-.6,3.12-1.57l2.32,4.63a27.11,27.11,0,0,0,28.09,14.72l72-10.29-.76,2.29.41-.06-23.34,70,3.86,1.29-.07.14.54.23-4.85,9.17-15.28,11.75S840.3,474.69,845,471.16s-16.46-18.81-16.46-18.81l-1.18-4.7,5.14-10.85,4.31,1.44L871.42,369h0l1.18-2.35L896,395.94l6.6-6.6.71.72.06-.06,9.15,9.26,6.63,18.1S953.87,381.46,951.92,375.92ZM770.24,241.31l-.46-.53a43.89,43.89,0,0,1-13.23,10.58v-3.85l.26-.23v-1.67A25.4,25.4,0,0,0,764,219.82l15.19,11.69Zm-51.61-43.4-.37-.37.37.36Zm-12.15,24.48q-.15,1.82-.12,3.68a38.28,38.28,0,0,0,.3,5.37c.78,6.22,3,11.3,7.6,13.72,4,2.1,6.63,2,8.31.55l.18.16a18,18,0,0,0,1.86,7,74,74,0,0,1-24.17-9.73s0,.31.06.87l-6.29-6.29Z" transform="translate(-169.23 -39.68)" fill="url(#00ef2338-96af-4b11-80f8-5a4c3164adbb)"/>
  <polygon points="651.99 378.3 641.71 397.71 657.69 404.56 667.97 382.87 651.99 378.3" fill="#fda57d"/>
  <path d="M826.92,444.24l1.14,4.57s20.55,14.84,16,18.27-48-18.27-48-18.27l14.84-11.42Z" transform="translate(-169.23 -39.68)" fill="#333"/>
  <polygon points="666.87 384.88 650.71 379.49 645.9 388.57 661.88 395.43 666.87 384.88" opacity="0.1"/>
  <polygon points="725 341.94 740.43 357.56 751.76 344.37 734.15 328.06 725 341.94" fill="#fda57d"/>
  <polygon points="734.78 328.82 725.63 342.7 731.48 348.63 743.35 336.76 734.78 328.82" opacity="0.1"/>
  <path d="M921,384.05l4.7.28s20.32-15.15,22.22-9.77-31.8,40.27-31.8,40.27l-6.44-17.58Z" transform="translate(-169.23 -39.68)" fill="#333"/>
  <path d="M774.66,267.22l23.72-27.44a5,5,0,0,0-.38-6.91l-51.61-39.14-6.28,9.78,40,30.79-17,18.62Z" transform="translate(-169.23 -39.68)" fill="#fda57d"/>
  <path d="M771.94,243.31l-8.3,9.1,11.53,14.31L787.06,253A34.67,34.67,0,0,1,771.94,243.31Z" transform="translate(-169.23 -39.68)" opacity="0.1"/>
  <path d="M705.78,273.18,678.93,253a8.16,8.16,0,0,1-2-10.87l40.18-44.71,8.22,8.22-27.75,34.64L714,256.75Z" transform="translate(-169.23 -39.68)" fill="#fda57d"/>
  <path d="M700.12,269.13l4.39,3.3L712.73,256l-10.32-10.32C702.63,249.18,703.25,262.76,700.12,269.13Z" transform="translate(-169.23 -39.68)" opacity="0.1"/>
  <polygon points="666.87 313.25 642.22 387.19 666.87 395.41 707.95 313.25 675.09 305.04 666.87 313.25" fill="#4d8af0"/>
  <polygon points="665.73 315.53 664.5 319.23 690.38 315.53 700.49 328.17 706.81 315.53 673.94 307.32 665.73 315.53" opacity="0.1"/>
  <path d="M717.07,197.51s10.53-13.57,18.52-4.44S723,203.34,723,203.34Z" transform="translate(-169.23 -39.68)" fill="#fda57d"/>
  <path d="M729.29,336.5l7.94,15.89a26.34,26.34,0,0,0,27.28,14.29l96.23-13.75L893.61,394l24.65-24.65-53-45.45a13.37,13.37,0,0,0-10.36-3.12Z" transform="translate(-169.23 -39.68)" fill="#4d8af0"/>
  <path d="M705.14,247.87s2.16,26.68-6.05,27.6l16.43,8.22s-.91,48.38,16.43,65.73c0,0,32-17.34,65.73-16.43s-8.22-32.86-8.22-32.86l-5.55-37.4,4.57-6.85a34.33,34.33,0,0,1-16-10.28h0c-4.57,5.71-15,13.45-24.12,13.45h-3.93c-13.53,0-27.92-2.73-39.32-10Z" transform="translate(-169.23 -39.68)" opacity="0.1"/>
  <path d="M703.62,245.59s2.16,26.68-6.05,27.6L714,281.4s-.91,48.38,16.43,65.73c0,0,32-17.34,65.73-16.43s-8.22-32.86-8.22-32.86l-5.55-37.4,4.57-6.85a34.33,34.33,0,0,1-16-10.28h0c-4.57,5.71-15,13.45-24.12,13.45h-3.93a74.73,74.73,0,0,1-39.32-11.17Z" transform="translate(-169.23 -39.68)" fill="#3ad29f"/>
  <path d="M725.26,238.07v9.17a17.53,17.53,0,0,0,5.13,12.39l19.51,19.51,6.5-13a16.29,16.29,0,0,0,1.72-7.29v-20.8Z" transform="translate(-169.23 -39.68)" fill="#fda57d"/>
  <path d="M725.51,248.63c0,.34,0,.67,0,1a24.64,24.64,0,0,0,32.83,0V239.46H725.51Z" transform="translate(-169.23 -39.68)" opacity="0.1"/>
  <circle cx="572.46" cy="190.17" r="24.65" fill="#fda57d"/>
  <path d="M725.26,222.9h41.08s16.43-16.43,0-24.65a56.24,56.24,0,0,0-11.15-3.94c-25-6.34-48.74,14.33-45.53,39.91.76,6,2.93,11,7.38,13.32C734.39,256.68,725.26,222.9,725.26,222.9Z" transform="translate(-169.23 -39.68)" opacity="0.1"/>
  <path d="M725.26,221.76h41.08s16.43-16.43,0-24.65a56.24,56.24,0,0,0-11.15-3.94c-25-6.34-48.74,14.33-45.53,39.91.76,6,2.93,11,7.38,13.32C734.39,255.54,725.26,221.76,725.26,221.76Z" transform="translate(-169.23 -39.68)" fill="#333"/>
  <ellipse cx="557.23" cy="190.49" rx="2.28" ry="4" fill="#fda57d"/>
  <polygon points="544.67 241.3 545.81 237.87 549.23 240.16 544.67 241.3" opacity="0.1"/>
  <path d="M788,297.85s-12.43,6-15.86,11.67" transform="translate(-169.23 -39.68)" opacity="0.1"/>
  <polygon points="613.17 220.75 612.03 224.17 609.74 220.75 613.17 220.75" opacity="0.1"/>
  <path d="M725.26,221.76h41.08s16.43-16.43,0-24.65a56.24,56.24,0,0,0-11.15-3.94c-25-6.34-48.74,14.33-45.53,39.91.76,6,2.93,11,7.38,13.32C734.39,255.54,725.26,221.76,725.26,221.76Z" transform="translate(-169.23 -39.68)" fill="#333"/>
  <script xmlns=""/>
</svg>

```

## File: static\src\js\payment_form.js

```javascript
/** @odoo-module **/

import { _t } from '@web/core/l10n/translation';

import PaymentForm from '@payment/js/payment_form';

PaymentForm.include({
    events: Object.assign({}, PaymentForm.prototype.events || {}, {
        'change input[name="o_donation_amount"]': '_updateAmount',
        'focus input[name="amount"]': '_updateAmount',
        'focus input[name="o_donation_amount"]': '_updateAmount',
    }),


    // #=== EVENT HANDLERS ===#

    /**
     * Update the amount in the payment context with the user input.
     *
     * @private
     * @param {Event} ev
     * @return {void}
     */
    _updateAmount(ev) {
        if (ev.target.value >= 0) {
            this.paymentContext.amount = ev.target.value;
            const otherAmountEl = this.el.querySelector("#other_amount");
            if (ev.target.id === "other_amount_value" && otherAmountEl) {
                otherAmountEl.value = ev.target.value;
            }
            if (ev.target.id === "other_amount" || ev.target.id === "other_amount_value") {
                this.el.querySelectorAll('input[name="o_donation_amount"][type="radio"]').forEach((radioEl) => {
                    radioEl.checked = false;
                });
            } else if (ev.target.name === "o_donation_amount" && otherAmountEl) {
                otherAmountEl.checked = false;
            }
        }
    },

    // #=== PAYMENT FLOW ===#

    /**
     * Perform some validations for donations before processing the payment flow.
     *
     * @override method from @payment/js/payment_form
     * @private
     * @param {string} providerCode - The code of the selected payment option's provider.
     * @param {number} paymentOptionId - The id of the selected payment option.
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @param {string} flow - The payment flow of the selected payment option.
     * @return {void}
     */
    async _initiatePaymentFlow(providerCode, paymentOptionId, paymentMethodCode, flow) {
        if (document.querySelector('.o_donation_payment_form')) {
            const errorFields = {};
            if (!this.el.querySelector('input[name="email"]').checkValidity()) {
                errorFields['email'] = _t("Email is invalid");
            }
            const mandatoryFields = {
                'name': _t('Name'),
                'email': _t('Email'),
                'country_id': _t('Country'),
            };
            for (const id in mandatoryFields) {
                const fieldEl = this.el.querySelector(`input[name="${id}"],select[name="${id}"]`);
                fieldEl.classList.remove('is-invalid');
                Popover.getOrCreateInstance(fieldEl)?.dispose();
                if (!fieldEl.value.trim()) {
                    errorFields[id] = _t("Field '%s' is mandatory", mandatoryFields[id]);
                }
            }
            if (Object.keys(errorFields).length) {
                for (const id in errorFields) {
                    const fieldEl = this.el.querySelector(
                        `input[name="${id}"],select[name="${id}"]`
                    );
                    fieldEl.classList.add('is-invalid');
                    Popover.getOrCreateInstance(fieldEl, {
                        content: errorFields[id],
                        placement: 'top',
                        trigger: 'hover',
                    });
                }
                this._displayErrorDialog(
                    _t("Payment processing failed"),
                    _t("Some information is missing to process your payment.")
                );
                this._enableButton();
                return;
            }
        }
        await this._super(...arguments);
    },

    /**
     * Add params used by the donation snippet for the RPC to the transaction route.
     *
     * @override method from @payment/js/payment_form
     * @private
     * @return {object} The extended transaction route params.
     */
    _prepareTransactionRouteParams() {
        const transactionRouteParams = this._super(...arguments);
        return document.querySelector('.o_donation_payment_form')
            ? {
            ...transactionRouteParams,
            partner_id: parseInt(this.paymentContext['partnerId']),
            currency_id: this.paymentContext['currencyId']
                    ? parseInt(this.paymentContext['currencyId']) : null,
            reference_prefix:this.paymentContext['referencePrefix']?.toString(),
            partner_details: {
                name: this.el.querySelector('input[name="name"]').value,
                email: this.el.querySelector('input[name="email"]').value,
                country_id: this.el.querySelector('select[name="country_id"]').value,
            },
            donation_comment: this.el.querySelector('#donation_comment').value,
            donation_recipient_email: this.el.querySelector(
                'input[name="donation_recipient_email"]'
            ).value,
        } : transactionRouteParams;
    },

});

```

## File: static\src\js\website_payment_donation.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';

publicWidget.registry.WebsitePaymentDonation = publicWidget.Widget.extend({
    selector: '.o_donation_payment_form',
    events: {
        'focus .o_amount_input': '_onFocusAmountInput',
        'change #donation_comment_checkbox': '_onChangeDonationComment'
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onFocusAmountInput(ev) {
        const otherAmountEl = this.el.querySelector("#other_amount");
        if (otherAmountEl) {
            otherAmountEl.checked = true;
        }
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onChangeDonationComment(ev) {
        const donationCommentEl = this.el.querySelector('#donation_comment');
        const checked = ev.currentTarget.checked;
        donationCommentEl.classList.toggle('d-none', !checked);
        if (!checked) {
            donationCommentEl.value = "";
        }
    },
});

```

## File: static\src\snippets\s_donation\000.js

```javascript
import { formatCurrency } from "@web/core/currency";
import { _t } from "@web/core/l10n/translation";
import publicWidget from '@web/legacy/js/public/public_widget';
import { rpc } from "@web/core/network/rpc";

const CUSTOM_BUTTON_EXTRA_WIDTH = 10;
let cachedCurrency;

publicWidget.registry.DonationSnippet = publicWidget.Widget.extend({
    selector: '.s_donation',
    disabledInEditableMode: false,
    events: {
        'click .s_donation_btn': '_onClickPrefilledButton',
        'click .s_donation_donate_btn': '_onClickDonateNowButton',
        'input #s_donation_range_slider': '_onInputRangeSlider',
    },

    /**
     * @override
     */
    async start() {
        await this._super(...arguments);
        this.$rangeSlider = this.$('#s_donation_range_slider');
        this.defaultAmount = this.el.dataset.defaultAmount;
        if (this.$rangeSlider.length) {
            this.$rangeSlider.val(this.defaultAmount);
            this._setBubble(this.$rangeSlider);
        }
        await this._displayCurrencies();
        const customButtonEl = this.el.querySelector("#s_donation_amount_input");
        if (customButtonEl) {
            const canvasEl = document.createElement("canvas");
            const context = canvasEl.getContext("2d");
            context.font = window.getComputedStyle(customButtonEl).font;
            const width = context.measureText(customButtonEl.placeholder).width;
            customButtonEl.style.maxWidth = `${Math.ceil(width) + CUSTOM_BUTTON_EXTRA_WIDTH}px`;
        }
    },
    /**
     * @override
     */
    destroy() {
        const customButtonEl = this.el.querySelector("#s_donation_amount_input");
        if (customButtonEl) {
            customButtonEl.style.maxWidth = "";
        }
        this.$el.find('.s_donation_currency').remove();
        this._deselectPrefilledButtons();
        this.$('.alert-danger').remove();
        this._super(...arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _deselectPrefilledButtons() {
        this.$('.s_donation_btn').removeClass('active');
    },
    /**
     * @private
     * @param {jQuery} $range
     */
    _setBubble($range) {
        const $bubble = this.$('.s_range_bubble');
        const val = $range.val();
        const min = $range[0].min || 0;
        const max = $range[0].max || 100;
        const newVal = Number(((val - min) * 100) / (max - min));
        const tipOffsetLow = 8 - (newVal * 0.16); // the range thumb size is 16px*16px. The '8' and the '0.16' are related to that 16px (50% and 1% of 16px)
        $bubble.contents().filter(function () {
            return this.nodeType === 3;
        }).replaceWith(val);

        // Sorta magic numbers based on size of the native UI thumb (source: https://css-tricks.com/value-bubbles-for-range-inputs/)
        $bubble[0].style.insetInlineStart = `calc(${newVal}% + (${tipOffsetLow}px))`;
    },
    /**
     * @private
     */
    _displayCurrencies() {
        return this._getCachedCurrency().then((result) => {
            // No need to recreate the elements if the currency is already set.
            if (this.currency === result) {
                return;
            }
            this.currency = result;
            this.$('.s_donation_currency').remove();
            const $prefilledButtons = this.$('.s_donation_btn, .s_range_bubble');
            $prefilledButtons.toArray().forEach((button) => {
                const before = result.position === "before";
                const $currencySymbol = document.createElement('span');
                $currencySymbol.innerText = result.symbol;
                $currencySymbol.classList.add('s_donation_currency', before ? "pe-1" : "ps-1");
                if (before) {
                    $(button).prepend($currencySymbol);
                } else {
                    $(button).append($currencySymbol);
                }
            });
        });
    },
    /**
     * @private
     */
    _getCachedCurrency() {
        return cachedCurrency
            ? Promise.resolve(cachedCurrency)
            : rpc("/website/get_current_currency").then((result) => {
                cachedCurrency = result;
                return result;
            });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onClickPrefilledButton(ev) {
        const $button = $(ev.currentTarget);
        this._deselectPrefilledButtons();
        $button.addClass('active');
        if (this.$rangeSlider.length) {
            this.$rangeSlider.val($button[0].dataset.donationValue);
            this._setBubble(this.$rangeSlider);
        }
    },
    /**
     * @private
     */
    _onClickDonateNowButton(ev) {
        if (this.editableMode) {
            return;
        };
        this.$('.alert-danger').remove();
        const $buttons = this.$('.s_donation_btn');
        const $selectedButton = $buttons.filter('.active');
        let amount = $selectedButton.length ? $selectedButton[0].dataset.donationValue : 0;
        if (this.el.dataset.displayOptions && !amount) {
            if (this.$rangeSlider.length) {
                amount = this.$rangeSlider.val();
            } else if ($buttons.length) {
                amount = parseFloat(this.$('#s_donation_amount_input').val());
                let errorMessage = '';
                const minAmount = parseFloat(this.el.dataset.minimumAmount);
                if (!amount) {
                    errorMessage = _t("Please select or enter an amount");
                } else if (amount < minAmount) {
                    errorMessage = _t(
                        "The minimum donation amount is %(amount)s",
                        {
                            amount: formatCurrency(minAmount, this.currency.id),
                        }
                    );
                }
                if (errorMessage) {
                    $(ev.currentTarget).before($('<p>', {
                        class: 'alert alert-danger',
                        text: errorMessage,
                    }));
                    return;
                }
            }
        }
        if (!amount) {
            amount = this.defaultAmount;
        }
        const $form = this.$('.s_donation_form');
        $('<input>').attr({type: 'hidden', name: 'amount', value: amount}).appendTo($form);
        $('<input>').attr({type: 'hidden', name: 'currency_id', value: this.currency.id}).appendTo($form);
        $('<input>').attr({type: 'hidden', name: 'csrf_token', value: odoo.csrf_token}).appendTo($form);
        $('<input>').attr({type: 'hidden', name: 'donation_options', value: JSON.stringify(this.el.dataset)}).appendTo($form);
        $form.submit();
    },
    /**
     * @private
     */
    _onInputRangeSlider(ev) {
        this._deselectPrefilledButtons();
        this._setBubble($(ev.currentTarget));
    },
});

export default {
    DonationSnippet: publicWidget.registry.DonationSnippet,
};

```

## File: static\src\snippets\s_donation\options.js

```javascript
/** @odoo-module **/

import { renderToElement } from "@web/core/utils/render";
import options from '@web_editor/js/editor/snippets.options';
import { _t } from "@web/core/l10n/translation";

options.registry.Donation = options.Class.extend({
    /**
     * @override
     */
    start() {
        this.defaultDescription = _t("Add a description here");
        return this._super(...arguments);
    },
    /**
     * @override
     */
    onBuilt() {
        this._rebuildPrefilledOptions();
        return this._super(...arguments);
    },
    /**
     * @override
     */
    cleanForSave() {
        if (!this.$target[0].dataset.descriptions) {
            this._updateDescriptions();
        }
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    async updateUI() {
        await this._super(...arguments);
        this._buildDescriptionsList();
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Show/hide options in the page.
     *
     * @see this.selectClass for parameters
     */
    displayOptions(previewMode, widgetValue, params) {
        this.$target[0].dataset.displayOptions = widgetValue;
        if (!widgetValue && this.$target[0].dataset.customAmount === "slider") {
            this.$target[0].dataset.customAmount = "freeAmount";
        } else if (widgetValue && !this.$target[0].dataset.prefilledOptions) {
            this.$target[0].dataset.customAmount = "slider";
        }
        this._rebuildPrefilledOptions();
    },
    /**
     * Add/remove prefilled buttons.
     *
     * @see this.selectClass for parameters
     */
    togglePrefilledOptions(previewMode, widgetValue, params) {
        this.$target[0].dataset.prefilledOptions = widgetValue;
        this.$el.find('.o_we_prefilled_options_list').toggleClass('d-none', !widgetValue);
        if (!widgetValue && this.$target[0].dataset.displayOptions) {
            this.$target[0].dataset.customAmount = "slider";
        }
        this._rebuildPrefilledOptions();
    },
    /**
     * Add/remove description of prefilled buttons.
     *
     * @see this.selectClass for parameters
     */
    toggleOptionDescription(previewMode, widgetValue, params) {
        this.$target[0].dataset.descriptions = widgetValue;
        this.renderListItems(false, this._buildPrefilledOptionsList());
    },
    /**
     * Select an amount input
     *
     * @see this.selectClass for parameters
     */
    selectAmountInput(previewMode, widgetValue, params) {
        this.$target[0].dataset.customAmount = widgetValue;
        this._rebuildPrefilledOptions();
    },
    /**
     * Apply the we-list on the target and rebuild the input(s)
     *
     * @see this.selectClass for parameters
     */
    renderListItems(previewMode, value, params) {
        const valueList = JSON.parse(value);
        const donationAmounts = [];
        delete this.$target[0].dataset.donationAmounts;
        valueList.forEach((value) => {
            donationAmounts.push(value.display_name);
        });
        this.$target[0].dataset.donationAmounts = JSON.stringify(donationAmounts);
        this._rebuildPrefilledOptions();
    },
    /**
     * Redraws the target whenever the list changes
     *
     * @see this.selectClass for parameters
     */
    listChanged(previewMode, value, params) {
        this._updateDescriptions();
        this._rebuildPrefilledOptions();
    },
    /**
     * @see this.selectClass for parameters
     */
    setMinimumAmount(previewMode, widgetValue, params) {
        this.$target[0].dataset.minimumAmount = widgetValue;
        const $rangeSlider = this.$('#s_donation_range_slider');
        const $amountInput = this.$('#s_donation_amount_input');
        if ($rangeSlider.length) {
            $rangeSlider[0].min = widgetValue;
        } else if ($amountInput.length) {
            $amountInput[0].min = widgetValue;
        }
    },
    /**
     * @see this.selectClass for parameters
     */
    setMaximumAmount(previewMode, widgetValue, params) {
        this.$target[0].dataset.maximumAmount = widgetValue;
        const $rangeSlider = this.$('#s_donation_range_slider');
        const $amountInput = this.$('#s_donation_amount_input');
        if ($rangeSlider.length) {
            $rangeSlider[0].max = widgetValue;
        } else if ($amountInput.length) {
            $amountInput[0].max = widgetValue;
        }
    },
    /**
     * @see this.selectClass for parameters
     */
    setSliderStep(previewMode, widgetValue, params) {
        this.$target[0].dataset.sliderStep = widgetValue;
        const $rangeSlider = this.$('#s_donation_range_slider');
        if ($rangeSlider.length) {
            $rangeSlider[0].step = widgetValue;
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _computeWidgetState(methodName, params) {
        switch (methodName) {
            case 'displayOptions': {
                return this.$target[0].dataset.displayOptions;
            }
            case 'togglePrefilledOptions': {
                return this.$target[0].dataset.prefilledOptions;
            }
            case 'toggleOptionDescription': {
                return this.$target[0].dataset.descriptions;
            }
            case 'selectAmountInput': {
                return this.$target[0].dataset.customAmount;
            }
            case 'renderListItems': {
                return this._buildPrefilledOptionsList();
            }
            case 'setMinimumAmount': {
                return this.$target[0].dataset.minimumAmount;
            }
            case 'setMaximumAmount': {
                return this.$target[0].dataset.maximumAmount;
            }
            case 'setSliderStep': {
                return this.$target[0].dataset.sliderStep;
            }
        }
        return this._super(...arguments);
    },
    /**
     * @override
     */
    async _computeWidgetVisibility(widgetName, params) {
        if (widgetName === 'free_amount_opt') {
            return !(this.$target[0].dataset.displayOptions && !this.$target[0].dataset.prefilledOptions);
        }
        return this._super(...arguments);
    },
    /**
     * @override
     */
    _renderCustomXML(uiFragment) {
        const list = document.createElement('we-list');
        list.dataset.dependencies = "pre_filled_opt";
        list.dataset.addItemTitle = _t("Add new pre-filled option");
        list.dataset.renderListItems = '';
        list.dataset.unsortable = 'true';
        list.dataset.inputType = 'number';
        list.dataset.defaultValue = 50;
        list.dataset.listChanged = '';
        $(uiFragment).find('we-checkbox[data-name="pre_filled_opt"]').after(list);
    },
    /**
     * Build the prefilled options list in the editor panel
     *
     * @private
     */
    _buildPrefilledOptionsList() {
        const amounts = JSON.parse(this.$target[0].dataset.donationAmounts);
        let valueList = amounts.map(amount => {
            return {
                id: amount,
                display_name: amount,
            };
        });
        return JSON.stringify(valueList);
    },
    /**
     * Add descriptions in the prefilled options list of the
     * editor panel.
     *
     * @private
     */
    _buildDescriptionsList() {
        if (this.$target[0].dataset.descriptions) {
            const $descriptions = this.$target.find('#s_donation_description_inputs > input');
            const $tableEl = this.$el.find('we-list table');
            $tableEl.find("tr").toArray().forEach((trEl, i) => {
                const $inputAmount = $(trEl).find('td').first();
                $inputAmount.addClass('w-25');
                const tdEl = document.createElement('td');
                const inputEl = document.createElement('input');
                inputEl.type = 'text';
                inputEl.value = $descriptions[i] ? $descriptions[i].value : this.defaultDescription;
                tdEl.classList.add('w-auto');
                tdEl.appendChild(inputEl);
                $(tdEl).insertAfter($inputAmount);
            });
            this._updateDescriptions();
        }
    },
    /**
     * Update descriptions in the input hidden.
     *
     * @private
     */
    _updateDescriptions() {
        const descriptionInputs = this.$target.find('#s_donation_description_inputs');
        descriptionInputs.empty();
        const descriptions = this.$el.find('we-list input[type=text]');
        descriptions.toArray().forEach((description) => {
            const inputEl = document.createElement('input');
            inputEl.type = 'hidden';
            inputEl.classList.add('o_translatable_input_hidden', 'd-block', 'mb-1', 'w-100');
            inputEl.name = 'donation_descriptions';
            inputEl.value = description.value;
            descriptionInputs[0].appendChild(inputEl);
        });
    },
    /**
     * Rebuild options in the DOM.
     *
     * @private
     */
    _rebuildPrefilledOptions() {
        const rebuild = this.$target[0].dataset.displayOptions;
        this.$target.find('.s_donation_prefilled_buttons').remove();
        const layout = this.$target[0].dataset.customAmount;
        const $slider = this.$target.find('.s_donation_range_slider_wrap');
        if (layout !== "slider" || !rebuild) {
            $slider.remove();
        }
        if (rebuild) {
            if (layout === "slider" && !$slider.length) {
                const sliderTemplate = $(renderToElement('website_payment.donation.slider', {
                    minimum_amount: this.$target[0].dataset.minimumAmount,
                    maximum_amount: this.$target[0].dataset.maximumAmount,
                    slider_step: this.$target[0].dataset.sliderStep,
                }));
                this.$target.find('.s_donation_donate_btn').before(sliderTemplate);
            }
            const prefilledOptions = this.$target[0].dataset.prefilledOptions;
            let donationAmounts = [];
            let showDescriptions = false;
            if (prefilledOptions) {
                donationAmounts = JSON.parse(this.$target[0].dataset.donationAmounts);
                showDescriptions = this.$target[0].dataset.descriptions;
                if (showDescriptions) {
                    const $descriptions = this.$target.find('#s_donation_description_inputs > input');
                    donationAmounts = donationAmounts.map((amount, i) => {
                        return {
                            value: amount,
                            description: $descriptions[i] ? $descriptions[i].value : this.defaultDescription,
                        };
                    });
                }
            }
            const $prefilledButtons = $(renderToElement(`website_payment.donation.prefilledButtons${showDescriptions ? 'Descriptions' : ''}`, {
                prefilled_buttons: donationAmounts,
                custom_input: layout === "freeAmount",
                minimum_amount: this.$target[0].dataset.minimumAmount,
            }));
            this.$target.find('#s_donation_description_inputs').after($prefilledButtons);
        }
    },
});

export default {
    Donation: options.registry.Donation,
};

```

## File: static\src\snippets\s_donation\options.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="website_payment.donation.prefilledButtons">
        <div class="s_donation_prefilled_buttons mb-2">
            <t t-foreach="prefilled_buttons" t-as="prefilled_button_value" t-key="prefilled_button_value_index">
                <button class="s_donation_btn btn btn-outline-primary btn-lg mb-2 me-1 o_not_editable"
                        type="button"
                        contenteditable="false"
                        t-att-data-donation-value="prefilled_button_value"
                        t-esc="prefilled_button_value"/>
            </t>
            <span t-if="custom_input" class="s_donation_btn s_donation_custom_btn btn btn-outline-primary btn-lg mb-2 me-1">
                <input id="s_donation_amount_input" type="number" t-att-min="minimum_amount" class="" placeholder="Custom Amount" aria-label="Amount"/>
            </span>
        </div>
    </t>
    <t t-name="website_payment.donation.prefilledButtonsDescriptions">
        <div class="s_donation_prefilled_buttons my-4">
            <t t-foreach="prefilled_buttons" t-as="prefilled_button" t-key="prefilled_button_value_index">
                <div class="s_donation_btn_description d-sm-flex align-items-center my-3 o_not_editable o_translate_mode_hidden" contenteditable="false">
                    <button class="s_donation_btn btn btn-outline-primary btn-lg me-3"
                            type="button"
                            t-att-data-donation-value="prefilled_button.value"
                            t-esc="prefilled_button.value"/>
                    <p class="s_donation_description mt-2 my-sm-auto text-muted fst-italic" t-esc="prefilled_button.description"></p>
                </div>
            </t>
            <div t-if="custom_input" class="d-sm-flex align-items-center my-3">
                <span class="s_donation_btn s_donation_custom_btn btn btn-outline-primary btn-lg">
                    <input id="s_donation_amount_input" type="number" t-att-min="minimum_amount" placeholder="Custom Amount" aria-label="Amount"/>
                </span>
            </div>
        </div>
    </t>
    <t t-name="website_payment.donation.slider">
        <div class="s_donation_range_slider_wrap mb-2 position-relative">
            <label for="s_donation_range_slider">Choose Your Amount</label>
            <input type="range" class="form-range" t-att-min="minimum_amount" t-att-max="maximum_amount" t-att-step="slider_step" id="s_donation_range_slider" contenteditable="false"/>
            <output class="s_range_bubble" contenteditable="false">25</output>
        </div>
    </t>
</templates>

```

## File: views\payment_form_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Insert the donation form inside the payment form's <form/> element. -->
    <template id="website_payment.payment_form" inherit_id="payment.form">
        <xpath expr="//div[@id='o_payment_form_options']" position="before">
            <div class="mb-3" t-if="is_donation">
                <t t-call="website_payment.donation_information"/>
            </div>
        </xpath>
    </template>

    <!-- Display of /donation/pay -->
    <template id="website_payment.donation_pay" name="Donation payment">
        <t t-call="portal.frontend_layout">
            <t t-set="page_title" t-value="'Donation'"/>
            <t t-set="additional_title"><t t-out="page_title"/></t>
            <div class="wrap">
                <div class="oe_structure" id="oe_structure_website_payment_donation_1"/>
                <div class="container mb-3">
                    <!-- Portal breadcrumb -->
                    <t t-call="payment.portal_breadcrumb"/>
                    <!-- Payment page -->
                    <div class="row">
                        <div class="col-lg-7">
                            <div t-if="not amount" class="alert alert-info">
                                There is nothing to pay.
                            </div>
                            <div t-elif="not currency" class="alert alert-warning">
                                <strong>Warning</strong> The currency is missing or incorrect.
                            </div>
                            <t t-else="" t-call="payment.form"/>
                        </div>
                    </div>
                </div>
                <div class="oe_structure" id="oe_structure_website_payment_donation_2"/>
            </div>
        </t>
    </template>

    <template id="website_payment.donation_information" name="Donation Information">
        <h3 class="o_page_header mt16 mb4">Donation</h3>
        <div class="row">
            <div t-attf-class="mb-3 #{error.get('name') and 'o_has_error' or ''} col-lg-12 div_name">
                <label class="col-form-label fw-bold" for="name">Name
                    <span class="s_website_form_mark"> *</span>
                </label>
                <input t-att-readonly="'1' if 'name' in partner_details and partner_id else None" type="text" name="name" t-attf-class="form-control #{error.get('name') and 'is-invalid' or ''}" t-att-value="partner_details.get('name')" />
            </div>
            <div class="w-100"/>
            <div t-attf-class="mb-3 #{error.get('email') and 'o_has_error' or ''} col-lg-6" id="div_email">
                <label class="col-form-label fw-bold" for="email">Email
                    <span class="s_website_form_mark"> *</span>
                </label>
                <input t-att-readonly="'1' if 'email' in partner_details and partner_id else None" type="email" name="email" t-attf-class="form-control #{error.get('email') and 'is-invalid' or ''}" t-att-value="partner_details.get('email')" />
            </div>
            <t t-set="country_id" t-value="partner_details.get('country_id')"/>
            <div t-attf-class="mb-3 #{error.get('country_id') and 'o_has_error' or ''} col-lg-6 div_country">
                <label class="col-form-label fw-bold" for="country_id">Country
                    <span class="s_website_form_mark"> *</span>
                </label>
                <select t-att-disabled="'1' if country_id and partner_id else None" id="country_id" name="country_id" t-attf-class="form-select #{error.get('country_id') and 'is-invalid' or ''}">
                    <option value="">Country...</option>
                    <t t-foreach="countries" t-as="c">
                        <option t-att-value="c.id" t-att-selected="c.id == (country_id or -1)">
                            <t t-out="c.name" />
                        </option>
                    </t>
                </select>
            </div>
            <div class="w-100"/>
            <div class="mb-3 col-lg-12 o_donation_payment_form">
                <div class="col-lg-6 px-0">
                    <label class="col-form-label fw-bold">Amount (<t t-out="currency.symbol"/>)</label>
                    <t t-set="donation_layout" t-value="donation_options.get('customAmount')"/>
                    <t t-set="prefilled_options" t-value="donation_options.get('prefilledOptions')"/>
                    <t t-if="prefilled_options">
                        <t t-foreach="donation_amounts" t-as="donation_amount">
                            <div class="form-check my-2">
                                <t t-set="is_checked" t-value="float(amount) == float(donation_amount)"/>
                                <t t-set="has_checked" t-value="has_checked or is_checked"/>
                                <input name="o_donation_amount"
                                       class="form-check-input"
                                       type="radio"
                                       t-attf-id="amount_#{donation_amount_index}"
                                       t-att-value="donation_amount"
                                       t-att-checked="is_checked or None"
                                />
                                <label class="form-check-label mt-0" t-attf-for="amount_#{donation_amount_index}">
                                    <t t-out="donation_amount"/>
                                    <span t-if="donation_options.get('descriptions')" class="text-muted fst-italic ms-1">
                                        - <t t-out="donation_descriptions[donation_amount_index]"/>
                                    </span>
                                </label>
                            </div>
                        </t>
                        <div t-attf-class="form-check my-2 #{not donation_layout and 'd-none' or ''}">
                            <input class="form-check-input" type="radio" id="other_amount" name="amount"
                                t-att-value="amount" t-att-checked="not has_checked or None"/>
                            <label class="form-check-label mt-0 d-block" for="other_amount"/>
                            <t t-call="website_payment.donation_input">
                                <t t-set="amount" t-value="not has_checked and amount or ''"/>
                            </t>
                        </div>
                    </t>
                    <t t-else="">
                        <t t-call="website_payment.donation_input"/>
                    </t>
                </div>
                <div class="col-lg-12 px-0">
                    <div class="form-check mt-3">
                        <input class="form-check-input" type="checkbox" value="" id="donation_comment_checkbox"/>
                        <label class="form-check-label" for="donation_comment_checkbox">Write us a comment</label>
                    </div>
                    <textarea class="form-control d-none mt-2" id="donation_comment" placeholder="Your comment"/>
                </div>
            </div>
            <input type="hidden" name="donation_recipient_email" t-att-value="donation_options.get('donationEmail')"/>
            <div class="w-100"/>
        </div>
        <h3 class="o_page_header mt16 mb4">Payment Details</h3>
    </template>

    <template id="website_payment.donation_input" name="Donation input">
        <input name="o_donation_amount"
               type="number"
               class="form-control o_amount_input"
               t-att-min="donation_options.get('minimumAmount')"
               t-att-max="donation_options.get('maximumAmount')"
               t-att-value="amount"
               placeholder="Custom Amount"
               id="other_amount_value"
        />
    </template>

</odoo>

```

## File: views\payment_provider.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="payment_provider_form" model="ir.ui.view">
            <field name="name">provider.form.inherit.website</field>
            <field name="model">payment.provider</field>
            <field name="inherit_id" ref="payment.payment_provider_form"/>
            <field name="arch" type="xml">
                <group name="payment_state" position='inside'>
                    <field name="website_id" options="{'no_open': True, 'no_create_edit': True}" groups="website.group_multi_website"/>
                </group>
            </field>
        </record>

    </data>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="20"/>
        <field name="inherit_id" ref="website.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <block id="website_info_settings" position="after">
                <block title="Shop - Payment" id="div_website_payment">
                    <setting
                        id="website_payment"
                        string="Activate Payments"
                        help="Support most payment methods; Visa, Mastercard, Maestro, Google Pay, Apple Pay, etc. as well as recurring charges."
                        documentation="/applications/finance/payment_providers.html"
                    >
                        <div class="content-group">
                            <div class="row" invisible="providers_state == 'other_than_paypal'">
                                <field name="providers_state" invisible="1"/>
                                <field name="is_stripe_supported_country" invisible="1"/>
                                <div class="oe_inline"
                                     invisible="not is_stripe_supported_country">
                                    <button string="Activate Stripe" name="action_activate_stripe" type="object" class="btn-primary"/>
                                </div>
                                <div class="oe_inline"
                                     title="Stripe Connect is not available in your country, please use another payment provider."
                                     invisible="is_stripe_supported_country">
                                    <button string="Activate Stripe" class="btn-primary" disabled=""/>
                                </div>
                                <button type="action"
                                        name="%(payment.action_payment_provider)d"
                                        string="View Alternatives"
                                        class="btn-link oe_inline"
                                        icon="oi-arrow-right"
                                />
                            </div>
                            <div invisible="providers_state != 'other_than_paypal'">
                                <button name="action_configure_first_provider"
                                        type="object"
                                        class="btn-primary col-auto"
                                >
                                    <field name="first_provider_label" nolabel="1" class="oe_inline"/>
                                </button>
                                <button type="action"
                                        name="%(payment.action_payment_provider)d"
                                        string="View other providers "
                                        class="btn-link col-auto"
                                        icon="oi-arrow-right"
                                />
                            </div>
                        </div>
                    </setting>
                </block>
            </block>
        </field>
    </record>
</odoo>

```

## File: views\snippets\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="remove_external_snippets" inherit_id="website.external_snippets">
    <xpath expr="//t[@t-install='website_payment']" position="replace"/>
</template>

<template id="snippets" inherit_id="website.snippets" name="Snippet Donation">
    <xpath expr="//t[@id='snippet_donation_hook']" position="replace">
        <t t-snippet="website_payment.s_donation" string="Donation" t-forbid-sanitize="form" group="contact_and_forms"/>
    </xpath>
    <xpath expr="//t[@id='snippet_donation_button_hook']" position="replace">
        <t t-snippet="website_payment.s_donation_button" string="Donation Button" t-thumbnail="/website/static/src/img/snippets_thumbs/s_donation_button.svg" t-forbid-sanitize="form"/>
    </xpath>
</template>

</odoo>

```

## File: views\snippets\s_donation.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_donation_button" name="Donation Button">
    <div class="s_donation"
            data-name="Donation Button"
            data-donation-email="info@yourcompany.example.com"
            data-custom-amount="freeAmount"
            t-att-data-display-options="display_options"
            data-prefilled-options="true"
            data-descriptions="true"
            data-donation-amounts='["10", "25", "50", "100"]'
            data-minimum-amount="5"
            data-maximum-amount="100"
            data-slider-step="5"
            data-default-amount="25">
        <form class="s_donation_form" action="/donation/pay" method="post" enctype="multipart/form-data">
            <span id="s_donation_description_inputs">
                <input type="hidden" class="o_translatable_input_hidden d-block mb-1 w-100" name="donation_descriptions" value="A year of cultural awakening."/>
                <input type="hidden" class="o_translatable_input_hidden d-block mb-1 w-100" name="donation_descriptions" value="Caring for a baby for 1 month."/>
                <input type="hidden" class="o_translatable_input_hidden d-block mb-1 w-100" name="donation_descriptions" value="One year in elementary school."/>
                <input type="hidden" class="o_translatable_input_hidden d-block mb-1 w-100" name="donation_descriptions" value="One year in high school."/>
            </span>
            <div class="s_donation_prefilled_buttons my-4">
                <div class="s_donation_btn_description d-sm-flex align-items-center my-3 o_not_editable o_translate_mode_hidden" contenteditable="false">
                    <button class="s_donation_btn btn btn-outline-primary btn-lg me-3" type="button" data-donation-value="10">
                        <span class="s_donation_currency pe-1">$</span>10
                    </button>
                    <p class="s_donation_description mt-2 my-sm-auto text-muted fst-italic">A year of cultural awakening.</p>
                </div>
                <div class="s_donation_btn_description d-sm-flex align-items-center my-3 o_not_editable o_translate_mode_hidden" contenteditable="false">
                    <button class="s_donation_btn btn btn-outline-primary btn-lg me-3" type="button" data-donation-value="25">
                        <span class="s_donation_currency pe-1">$</span>25
                    </button>
                    <p class="s_donation_description mt-2 my-sm-auto text-muted fst-italic">Caring for a baby for 1 month.</p>
                </div>
                <div class="s_donation_btn_description d-sm-flex align-items-center my-3 o_not_editable o_translate_mode_hidden" contenteditable="false">
                    <button class="s_donation_btn btn btn-outline-primary btn-lg me-3" type="button" data-donation-value="50">
                        <span class="s_donation_currency pe-1">$</span>50
                    </button>
                    <p class="s_donation_description mt-2 my-sm-auto text-muted fst-italic">One year in elementary school.</p>
                </div>
                <div class="s_donation_btn_description d-sm-flex align-items-center my-3 o_not_editable o_translate_mode_hidden" contenteditable="false">
                    <button class="s_donation_btn btn btn-outline-primary btn-lg me-3" type="button" data-donation-value="100">
                        <span class="s_donation_currency pe-1">$</span>100
                    </button>
                    <p class="s_donation_description mt-2 my-sm-auto text-muted fst-italic">One year in high school.</p>
                </div>
                <div class="d-sm-flex align-items-center my-3">
                    <span class="s_donation_btn s_donation_custom_btn btn btn-outline-primary btn-lg">
                        <span class="s_donation_currency pe-1">$</span>
                        <input id="s_donation_amount_input" type="number" placeholder="Custom Amount" aria-label="Amount" min="5" style="max-width: 156px;"/>
                    </span>
                </div>
            </div>
            <a href="#" type="button" class="s_donation_donate_btn btn btn-secondary btn-lg mb-2">Donate Now</a>
        </form>
    </div>
</template>

<template id="s_donation" name="Donation">
    <section class="pt32 pb32 o_cc o_cc1">
        <div class="container">
            <div class="row align-items-center">
                <div class="col-lg-7 pt16 pb16">
                    <h2>Make a Donation</h2>
                    <p>Small or large, your contribution is essential.</p>
                    <t t-snippet-call="website_payment.s_donation_button">
                        <t t-set="display_options" t-value="'true'"/>
                    </t>
                </div>
                <div class="col-lg-5 pt16 pb16 d-none d-md-block">
                    <img src="/web_editor/shape/website_payment/s_donation_gift.svg?c1=o-color-1" class="img img-fluid mx-auto" style="width: 75%;" alt=""/>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_donation_options"  inherit_id="website.snippet_options">
    <xpath expr="." position="inside">
        <div data-js="Donation" data-selector=".s_donation">
            <we-input class="o_we_large" string="Recipient Email" data-select-data-attribute=""
                data-attribute-name="donationEmail"/>
            <we-checkbox string="Display Options"
                    data-name="display_options_opt"
                    data-display-options="true"
                    data-no-preview="true">
            </we-checkbox>
            <we-checkbox string="Pre-filled Options"
                    data-name="pre_filled_opt"
                    data-toggle-prefilled-options="true"
                    data-dependencies="!no_input_opt"
                    data-no-preview="true">
            </we-checkbox>
            <we-checkbox string="Descriptions"
                         class="o_we_sublevel_1"
                         data-toggle-option-description="true"
                         data-dependencies="pre_filled_opt"
                         data-no-preview="true">
            </we-checkbox>
            <we-select string="Custom Amount" data-no-preview="true">
                <we-button data-name="free_amount_opt" data-select-amount-input="freeAmount">Input</we-button>
                <we-button data-name="slider_opt" data-select-amount-input="slider" data-dependencies="display_options_opt">Slider</we-button>
                <we-button data-name="no_input_opt" data-select-amount-input="" data-dependencies="pre_filled_opt">None</we-button>
            </we-select>
            <we-input string="Minimum" class="o_we_sublevel_1"
                      data-step="1" data-set-minimum-amount="" data-dependencies="!no_input_opt"/>
            <we-input string="Maximum" class="o_we_sublevel_1"
                      data-step="1" data-set-maximum-amount="" data-dependencies="slider_opt"/>
            <we-input string="Step" class="o_we_sublevel_1"
                      data-step="1" data-set-slider-step="" data-dependencies="slider_opt"/>
            <we-input string="Default Amount" data-step="1" data-attribute-default-value="25"
                data-select-data-attribute="" data-attribute-name="defaultAmount"/>
        </div>
    </xpath>
</template>

<record id="website_payment.s_donation_000_js" model="ir.asset">
    <field name="name">Donation 000 JS</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">website_payment/static/src/snippets/s_donation/000.js</field>
</record>

<record id="website_payment.s_donation_000_scss" model="ir.asset">
    <field name="name">Donation 000 SCSS</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">website_payment/static/src/snippets/s_donation/000.scss</field>
</record>

</odoo>

```

