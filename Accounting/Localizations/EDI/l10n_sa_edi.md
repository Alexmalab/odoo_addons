# Odoo Module: l10n_sa_edi

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
from . import models, wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Saudi Arabia - E-invoicing',
    'icon': '/l10n_sa/static/description/icon.png',
    'version': '0.1',
    'depends': [
        'account_edi_ubl_cii',
        'account_debit_note',
        'l10n_sa',
        'base_vat'
    ],
    'author': 'Odoo S.A.',
    'summary': """
        E-Invoicing, Universal Business Language
    """,
    'description': """
E-invoice implementation for Saudi Arabia; Integration with ZATCA
    """,
    'category': 'Accounting/Localizations/EDI',
    'license': 'LGPL-3',
    'data': [
        'security/ir.model.access.csv',
        'data/account_edi_format.xml',
        'data/ubl_21_zatca.xml',
        'data/res_country_data.xml',
        'wizard/l10n_sa_edi_otp_wizard.xml',
        'wizard/account_move_reversal_views.xml',
        'views/account_tax_views.xml',
        'views/account_journal_views.xml',
        'views/res_partner_views.xml',
        'views/res_company_views.xml',
        'views/res_config_settings_view.xml',
        'views/report_invoice.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'l10n_sa_edi/static/src/scss/form_view.scss',
        ]
    }
}

```

## File: data\account_edi_format.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="edi_sa_zatca" model="account.edi.format">
            <field name="name">ZATCA (Saudi Arabia)</field>
            <field name="code">sa_zatca</field>
        </record>

    </data>
</odoo>

```

## File: data\neutralize.sql

```sql
-- disable l10n_sa_edi
UPDATE res_company
SET l10n_sa_api_mode = 'sandbox';
```

## File: data\pre-hash_invoice.xsl

```xsl
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
                xmlns:xs="http://www.w3.org/2001/XMLSchema"
                xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                xmlns:ext="urn:oasis:names:specification:ubl:schema:xsd:CommonExtensionComponents-2"
                exclude-result-prefixes="xs"
                version="2.0">
    <xsl:output omit-xml-declaration="yes" indent="no"/>
    <xsl:template match="node() | @*">
        <xsl:copy>
            <xsl:apply-templates select="node() | @*"/>
        </xsl:copy>
    </xsl:template>
    <xsl:template match="//*[local-name()='Invoice']//*[local-name()='UBLExtensions']"></xsl:template>
    <xsl:template match="//*[local-name()='AdditionalDocumentReference'][cbc:ID[normalize-space(text()) = 'QR']]"></xsl:template>
    <xsl:template match="//*[local-name()='Invoice']//*[local-name()='Signature']"></xsl:template>
</xsl:stylesheet>

```

## File: data\res_country_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sa_partner_address_form" model="ir.ui.view">
        <field name="name">sa.partner.form.address</field>
        <field name="model">res.partner</field>
        <field name="priority" eval="900"/>
        <field name="arch" type="xml">
            <form>
                <div class="o_address_format">
                    <field name="parent_id" invisible="1"/>
                    <field name="type" invisible="1"/>
                    <field name="street" placeholder="Street" class="o_address_street"
                           attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="street2" placeholder="Neighborhood" class="o_address_street"
                            attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="city" placeholder="City" class="o_address_city"
                           attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="state_id" class="o_address_state" placeholder="State..." options='{"no_open": True}'
                           attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="zip" placeholder="ZIP" class="o_address_zip"
                           attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="country_id" placeholder="Country" class="o_address_country" options='{"no_open": True, "no_create": True}'
                           attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="l10n_sa_edi_building_number" placeholder="Building Number"
                           class="o_address_building_number" options='{"no_open": True, "no_create": True}'
                           attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="l10n_sa_edi_plot_identification" placeholder="Plot Identification"
                           class="o_address_plot_identification" options='{"no_open": True, "no_create": True}'
                           attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                </div>
            </form>
        </field>
    </record>
    <record id="base.sa" model="res.country">
        <field name="address_view_id" ref="sa_partner_address_form" />
        <field name="address_format" eval="'%(street)s\n%(street2)s\n%(city)s %(state_code)s %(zip)s\n%(l10n_sa_edi_building_number)s %(l10n_sa_edi_plot_identification)s\n%(country_name)s'"/>
    </record>
</odoo>

```

## File: data\ubl_21_zatca.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <template id="ubl_21_PaymentMeansType_zatca" inherit_id="account_edi_ubl_cii.ubl_20_PaymentMeansType" primary="True">
            <xpath expr="//*[local-name()='InstructionID']" position="after">
                <cbc:InstructionNote xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                                     xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                                     t-out="vals['adjustment_reason']"/>
            </xpath>
        </template>

        <template id="export_sa_zatca_ubl_extensions">
            <ext:UBLExtensions xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                               xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                               xmlns:ext="urn:oasis:names:specification:ubl:schema:xsd:CommonExtensionComponents-2">
                <ext:UBLExtension>
                    <ext:ExtensionURI>urn:oasis:names:specification:ubl:dsig:enveloped:xades</ext:ExtensionURI>
                    <ext:ExtensionContent>
                        <sig:UBLDocumentSignatures
                                xmlns:sac="urn:oasis:names:specification:ubl:schema:xsd:SignatureAggregateComponents-2"
                                xmlns:sbc="urn:oasis:names:specification:ubl:schema:xsd:SignatureBasicComponents-2"
                                xmlns:sig="urn:oasis:names:specification:ubl:schema:xsd:CommonSignatureComponents-2">
                            <sac:SignatureInformation>
                                <cbc:ID>urn:oasis:names:specification:ubl:signature:1</cbc:ID>
                                <sbc:ReferencedSignatureID>urn:oasis:names:specification:ubl:signature:Invoice
                                </sbc:ReferencedSignatureID>
                                <ds:Signature Id="signature" xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
                                    <ds:SignedInfo>
                                        <ds:CanonicalizationMethod
                                                Algorithm="http://www.w3.org/2006/12/xml-c14n11"/>
                                        <ds:SignatureMethod
                                                Algorithm="http://www.w3.org/2001/04/xmldsig-more#ecdsa-sha256"/>
                                        <ds:Reference Id="invoiceSignedData" URI="">
                                            <ds:Transforms>
                                                <ds:Transform Algorithm="http://www.w3.org/TR/1999/REC-xpath-19991116">
                                                    <ds:XPath>not(//ancestor-or-self::ext:UBLExtensions)</ds:XPath>
                                                </ds:Transform>
                                                <ds:Transform Algorithm="http://www.w3.org/TR/1999/REC-xpath-19991116">
                                                    <ds:XPath>not(//ancestor-or-self::cac:Signature)</ds:XPath>
                                                </ds:Transform>
                                                <ds:Transform Algorithm="http://www.w3.org/TR/1999/REC-xpath-19991116">
                                                    <ds:XPath>
                                                        not(//ancestor-or-self::cac:AdditionalDocumentReference[cbc:ID='QR'])
                                                    </ds:XPath>
                                                </ds:Transform>
                                                <ds:Transform Algorithm="http://www.w3.org/2006/12/xml-c14n11"/>
                                            </ds:Transforms>
                                            <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
                                            <!-- b64encoded SHA256 digest of document -->
                                            <ds:DigestValue/>
                                        </ds:Reference>
                                        <ds:Reference Type="http://www.w3.org/2000/09/xmldsig#SignatureProperties"
                                                      URI="#xadesSignedProperties">
                                            <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
                                            <ds:DigestValue/>
                                        </ds:Reference>
                                    </ds:SignedInfo>
                                    <ds:SignatureValue/>
                                    <ds:KeyInfo>
                                        <ds:X509Data>
                                            <ds:X509Certificate/>
                                        </ds:X509Data>
                                    </ds:KeyInfo>
                                    <ds:Object>
                                        <xades:QualifyingProperties xmlns:xades="http://uri.etsi.org/01903/v1.3.2#"
                                                                    Target="signature">
                                            <xades:SignedProperties xmlns:xades="http://uri.etsi.org/01903/v1.3.2#"
                                                                    Id="xadesSignedProperties">
                                                <xades:SignedSignatureProperties>
                                                    <xades:SigningTime/>
                                                    <xades:SigningCertificate>
                                                        <xades:Cert>
                                                            <xades:CertDigest>
                                                                <ds:DigestMethod
                                                                        xmlns:ds="http://www.w3.org/2000/09/xmldsig#"
                                                                        Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
                                                                <ds:DigestValue
                                                                        xmlns:ds="http://www.w3.org/2000/09/xmldsig#"/>
                                                            </xades:CertDigest>
                                                            <xades:IssuerSerial>
                                                                <ds:X509IssuerName
                                                                        xmlns:ds="http://www.w3.org/2000/09/xmldsig#"/>
                                                                <ds:X509SerialNumber
                                                                        xmlns:ds="http://www.w3.org/2000/09/xmldsig#"/>
                                                            </xades:IssuerSerial>
                                                        </xades:Cert>
                                                    </xades:SigningCertificate>
                                                </xades:SignedSignatureProperties>
                                            </xades:SignedProperties>
                                        </xades:QualifyingProperties>
                                    </ds:Object>
                                </ds:Signature>
                            </sac:SignatureInformation>
                        </sig:UBLDocumentSignatures>
                    </ext:ExtensionContent>
                </ext:UBLExtension>
            </ext:UBLExtensions>
        </template>

        <template id="export_sa_zatca_ubl_signed_properties">
            <xades:SignedProperties xmlns:xades="http://uri.etsi.org/01903/v1.3.2#" Id="xadesSignedProperties">
                <xades:SignedSignatureProperties>
                    <xades:SigningTime t-out="signing_time"/>
                    <xades:SigningCertificate>
                        <xades:Cert>
                            <xades:CertDigest>
                                <ds:DigestMethod xmlns:ds="http://www.w3.org/2000/09/xmldsig#"
                                                 Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
                                <ds:DigestValue xmlns:ds="http://www.w3.org/2000/09/xmldsig#"
                                                t-out="public_key_hashing"/>
                            </xades:CertDigest>
                            <xades:IssuerSerial>
                                <ds:X509IssuerName xmlns:ds="http://www.w3.org/2000/09/xmldsig#" t-out="issuer_name"/>
                                <ds:X509SerialNumber xmlns:ds="http://www.w3.org/2000/09/xmldsig#"
                                                     t-out="serial_number"/>
                            </xades:IssuerSerial>
                        </xades:Cert>
                    </xades:SigningCertificate>
                </xades:SignedSignatureProperties>
            </xades:SignedProperties>
        </template>

        <template id="ubl_21_InvoiceLineType_zatca" inherit_id="account_edi_ubl_cii.ubl_20_InvoiceLineType" primary="True">
            <!-- Remove SellersItemIdentification if None -->
            <xpath expr="//*[local-name()='SellersItemIdentification']" position="replace">
                <cac:SellersItemIdentification t-if="item_vals['sellers_item_identification_vals']['id']"
                                               xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                                               xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                                               xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                    <cbc:ID t-out="item_vals['sellers_item_identification_vals']['id']"/>
                </cac:SellersItemIdentification>
            </xpath>
            <xpath expr="//*[local-name()='CreditedQuantity']" position="replace"/>
            <xpath expr="//*[local-name()='InvoicedQuantity']" position="replace">
                <cbc:InvoicedQuantity
                        xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                        xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                        t-att="vals.get('invoiced_quantity_attrs', {})"
                        t-out="vals['invoiced_quantity']"/>
            </xpath>
            <xpath expr="//*[local-name()='LineExtensionAmount']" position="after">
                <cac:DocumentReference
                        t-if="vals.get('prepayment_vals', {})"
                        xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                        xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                        xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                    <cbc:ID t-out="vals['prepayment_vals']['prepayment_id']"/>
                    <cbc:IssueDate t-esc="vals['prepayment_vals']['issue_date'].strftime('%Y-%m-%d')"/>
                    <cbc:IssueTime t-esc="vals['prepayment_vals']['issue_date'].strftime('%H:%M:%S')"/>
                    <cbc:DocumentTypeCode t-out="vals['prepayment_vals']['document_type_code']"/>
                </cac:DocumentReference>
            </xpath>
        </template>

        <template id="ubl_21_TaxTotalType_zatca" inherit_id="account_edi_ubl_cii.ubl_20_TaxTotalType" primary="True">
            <xpath expr="//*[local-name()='TaxAmount']" position="after">
                <cbc:RoundingAmount xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                                    xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                                    t-att-currencyID="vals['currency'].name"
                                    t-esc="format_float(vals.get('total_amount_sa'), vals['currency_dp'])"/>
            </xpath>
        </template>

        <template id="ubl_21_PartyType_zatca" inherit_id="account_edi_ubl_cii.ubl_20_PartyType" primary="True">
            <xpath expr="//*[local-name()='PartyIdentification']" position="replace">
                <cac:PartyIdentification t-if="party_vals.get('id')"
                                         xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                                         xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                                         xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                    <cbc:ID t-att="party_vals.get('id_attrs', {})" t-out="party_vals['id']"/>
                </cac:PartyIdentification>
            </xpath>
        </template>

        <template id="ubl_21_AddressType_zatca" inherit_id="account_edi_ubl_cii.ubl_20_AddressType" primary="True">
            <!-- AdditionalStreetName causes the Validation SDK to crash, so it has to be removed -->
            <xpath expr="//*[local-name()='AdditionalStreetName']" position="replace"/>
            <xpath expr="//*[local-name()='StreetName']" position="after">
                <t xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                   xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                   xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                    <!--  Add Building number in compliance with rules KSA-17 (seller)/ KSA-18 (customer)  -->
                    <cbc:BuildingNumber t-out="vals.get('building_number')"/>
                    <!--  Add Plot identification in compliance with rules KSA-23 (seller)/ KSA-19 (customer)  -->
                    <cbc:PlotIdentification t-out="vals.get('plot_identification')"/>
                    <!--  Add Neighborhood in compliance with rules KSA-3 (seller)/ KSA-4 (customer)  -->
                    <cbc:CitySubdivisionName t-out="vals.get('neighborhood')"/>
                </t>
            </xpath>
        </template>

        <template id="ubl_21_InvoiceType_zatca" inherit_id="account_edi_ubl_cii.ubl_21_InvoiceType" primary="True">

            <!--    For ZATCA, we do not use CreditNoteTypeCode or DebitNoteTypeCode tags. We always use InvoiceTypeCode. -->
            <xpath expr="//*[local-name()='CreditNoteTypeCode']" position="replace"/>
            <xpath expr="//*[local-name()='InvoiceTypeCode']" position="replace">
                <cbc:InvoiceTypeCode xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                                     xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                                     xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                                     t-att="vals['invoice_type_code_attrs']"
                                     t-out="vals['invoice_type_code']"/>
            </xpath>

            <!--    For ZATCA, we do not use CreditNoteLine or DebitNoteLine tags. We always use InvoiceLine. -->
            <xpath expr="//*[local-name()='CreditNoteLine']" position="replace"/>
            <xpath expr="//*[local-name()='InvoiceLine']" position="replace">
                <cac:InvoiceLine xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                                 xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                                 xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                    <t t-call="{{InvoiceLineType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:InvoiceLine>
            </xpath>

            <!-- Remove Order Reference if None -->
            <xpath expr="//*[local-name()='OrderReference']" position="replace">
                <cac:OrderReference t-if="vals.get('order_reference')"
                                    xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                                    xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                                    xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                    <cbc:ID t-out="vals['order_reference']"/>
                </cac:OrderReference>
            </xpath>

            <!-- Add Invoice UUID in compliance with rule BR-KSA-03 -->
            <xpath expr="//*[local-name()='ID']" position="after">
                <cbc:UUID xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                          xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                          t-esc="invoice.l10n_sa_uuid"/>
            </xpath>

            <!-- Add Invoice Issue Time in compliance with rule KSA-25 -->
            <xpath expr="//*[local-name()='IssueDate']" position="replace">
                <t xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                   xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                    <cbc:IssueDate t-esc="vals['issue_date'].strftime('%Y-%m-%d')"/>
                    <cbc:IssueTime t-esc="vals['issue_date'].strftime('%H:%M:%S')"/>
                </t>
            </xpath>

            <!-- Add Tax Currency Code in compliance with rules BR-KSA-EN16931-02 and BR-KSA-68 -->
            <xpath expr="//*[local-name()='DocumentCurrencyCode']" position="after">
                <cbc:TaxCurrencyCode xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                                     xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                                     t-esc="invoice.company_currency_id.name"/>
            </xpath>

            <!-- Add Previous Invoice Hash & Invoice Counter Value -->
            <xpath expr="//*[local-name()='BillingReference']" position="after">
                <t xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                   xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                   xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                    <!-- Add QR Code in compliance with rule BR-KSA-27 -->
                    <cac:AdditionalDocumentReference t-if="invoice._l10n_sa_is_simplified()">
                        <cbc:ID>QR</cbc:ID>
                        <cac:Attachment>
                            <cbc:EmbeddedDocumentBinaryObject mimeCode="text/plain">N/A</cbc:EmbeddedDocumentBinaryObject>
                        </cac:Attachment>
                    </cac:AdditionalDocumentReference>
                    <!-- Add Previous Invoice Hash in compliance with rule BR-KSA-61 -->
                    <cac:AdditionalDocumentReference>
                        <cbc:ID>PIH</cbc:ID>
                        <cac:Attachment>
                            <cbc:EmbeddedDocumentBinaryObject mimeCode="text/plain"
                                                              t-out="vals['previous_invoice_hash']"/>
                        </cac:Attachment>
                    </cac:AdditionalDocumentReference>
                    <!-- Add Invoice Counter Value in compliance with rules BR-KSA-33 and BR-KSA-34 -->
                    <cac:AdditionalDocumentReference>
                        <cbc:ID>ICV</cbc:ID>
                        <cbc:UUID t-out="invoice.l10n_sa_chain_index"/>
                    </cac:AdditionalDocumentReference>
                    <!-- Add Signature references in compliance with rules BR-KSA-29 and BR-KSA-30 -->
                    <cac:Signature t-if="invoice._l10n_sa_is_simplified()">
                        <cbc:ID>urn:oasis:names:specification:ubl:signature:Invoice</cbc:ID>
                        <cbc:SignatureMethod>urn:oasis:names:specification:ubl:dsig:enveloped:xades</cbc:SignatureMethod>
                    </cac:Signature>
                </t>
            </xpath>
        </template>

    </data>
</odoo>

```

## File: models\account_edi_document.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import models


class AccountEdiDocument(models.Model):
    _inherit = 'account.edi.document'

    def _prepare_jobs(self):
        """
        Override to achieve the following:

        If there is a job to process that may already be part of the chain (posted invoice that timed out),
        Moves it at the beginning of the list.
        """
        jobs = super()._prepare_jobs()
        if len(jobs) > 1:
            move_first_index = 0
            for index, job in enumerate(jobs):
                documents = job['documents']
                if any(d.edi_format_id.code == 'sa_zatca' and d.state == 'to_send' and d.move_id.l10n_sa_chain_index for d in documents):
                    move_first_index = index
                    break
            jobs = [jobs[move_first_index]] + jobs[:move_first_index] + jobs[move_first_index + 1:]

        return jobs

```

## File: models\account_edi_format.py

```python
import logging

from markupsafe import Markup
from hashlib import sha256
from base64 import b64decode, b64encode
from lxml import etree
from datetime import datetime
from odoo import models, fields, _, api
from odoo.exceptions import UserError
from cryptography.hazmat.primitives.serialization import load_pem_private_key
from cryptography.hazmat.primitives.asymmetric.ec import ECDSA
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.backends import default_backend
from cryptography.x509 import load_der_x509_certificate

_logger = logging.getLogger(__name__)


class AccountEdiFormat(models.Model):
    _inherit = 'account.edi.format'

    """
        Once the journal has been successfully onboarded, we can clear/report invoices through the ZATCA API:
            A) STANDARD Invoice:
                Make a call to the Clearance API '/invoices/clearance/single'.
                This will validate the invoice, sign it and apply a QR code then return the result.
            B) SIMPLIFIED Invoice:
                Make a call to the Reporting API '/invoices/reporting/single'.
                This will validate the invoice then return the result.
        The X509 Certificate and password from the PCSID API need to be provided in the request headers.
    """

    # ====== Helper Functions =======

    def _l10n_sa_get_zatca_datetime(self, timestamp):
        return fields.Datetime.context_timestamp(self.with_context(tz='Asia/Riyadh'), timestamp)

    def _l10n_sa_xml_node_content(self, root, xpath, namespaces=None):
        namespaces = namespaces or self.env['account.edi.xml.ubl_21.zatca']._l10n_sa_get_namespaces()
        return etree.tostring(root.xpath(xpath, namespaces=namespaces)[0], with_tail=False,
                              encoding='utf-8', method='xml')

    # ====== Xades Signing =======

    @api.model
    def _l10n_sa_get_digital_signature(self, company_id, invoice_hash):
        """
            Generate an ECDSA SHA256 digital signature for the XML eInvoice
        """
        decoded_hash = b64decode(invoice_hash).decode()
        private_key = load_pem_private_key(company_id.sudo().l10n_sa_private_key, password=None, backend=default_backend())
        signature = private_key.sign(decoded_hash.encode(), ECDSA(hashes.SHA256()))
        return b64encode(signature)

    def _l10n_sa_calculate_signed_properties_hash(self, issuer_name, serial_number, signing_time, public_key):
        """
            Calculate the SHA256 value of the SignedProperties XML node. The algorithm used by ZATCA expects the indentation
            of the nodes to start with 40 spaces, except for the root SignedProperties node.
        """
        signed_properties = etree.fromstring(self.env['ir.qweb']._render('l10n_sa_edi.export_sa_zatca_ubl_signed_properties', {
            'issuer_name': issuer_name,
            'serial_number': serial_number,
            'signing_time': signing_time,
            'public_key_hashing': public_key,
        }))
        etree.indent(signed_properties, space='    ')
        signed_properties_split = etree.tostring(signed_properties).decode().split('\n')
        signed_properties_final = ""
        for index, line in enumerate(signed_properties_split):
            if index == 0:
                signed_properties_final += line
            else:
                signed_properties_final += (' ' * 36) + line
            if index != len(signed_properties_final) - 1:
                signed_properties_final += '\n'
        signed_properties_final = etree.tostring(etree.fromstring(signed_properties_final))
        return b64encode(sha256(signed_properties_final).hexdigest().encode()).decode()

    def _l10n_sa_sign_xml(self, xml_content, certificate_str, signature):
        """
            Function that signs XML content of a UBL document with a provided B64 encoded X509 certificate
        """
        root = etree.fromstring(xml_content)
        etree.indent(root, space='    ')

        def _set_content(xpath, content):
            node = root.xpath(xpath)[0]
            node.text = content

        b64_decoded_cert = b64decode(certificate_str)
        x509_certificate = load_der_x509_certificate(b64decode(b64_decoded_cert.decode()), default_backend())

        issuer_name = ', '.join([s.rfc4514_string() for s in x509_certificate.issuer.rdns[::-1]])
        serial_number = str(x509_certificate.serial_number)
        signing_time = self._l10n_sa_get_zatca_datetime(datetime.now()).strftime('%Y-%m-%dT%H:%M:%SZ')
        public_key_hashing = b64encode(sha256(b64_decoded_cert).hexdigest().encode()).decode()

        signed_properties_hash = self._l10n_sa_calculate_signed_properties_hash(issuer_name, serial_number,
                                                                                signing_time, public_key_hashing)

        _set_content("//*[local-name()='X509IssuerName']", issuer_name)
        _set_content("//*[local-name()='X509SerialNumber']", serial_number)
        _set_content("//*[local-name()='SignedSignatureProperties']/*[local-name()='SigningTime']", signing_time)
        _set_content("//*[local-name()='SignedSignatureProperties']//*[local-name()='DigestValue']", public_key_hashing)

        prehash_content = etree.tostring(root)
        invoice_hash = self.env['account.edi.xml.ubl_21.zatca']._l10n_sa_generate_invoice_xml_hash(prehash_content,
                                                                                                   'digest')

        _set_content("//*[local-name()='SignatureValue']", signature)
        _set_content("//*[local-name()='X509Certificate']", b64_decoded_cert.decode())
        _set_content("//*[local-name()='SignatureInformation']//*[local-name()='DigestValue']", invoice_hash)
        _set_content("//*[@URI='#xadesSignedProperties']/*[local-name()='DigestValue']", signed_properties_hash)

        return etree.tostring(root, with_tail=False)

    def _l10n_sa_assert_clearance_status(self, invoice, clearance_data):
        """
            Assert Clearance status. To be overridden in case there are any other cases to be accounted for
        """
        mode = 'reporting' if invoice._l10n_sa_is_simplified() else 'clearance'
        if mode == 'clearance' and clearance_data.get('clearanceStatus', '') != 'CLEARED':
            return {'error': _("Invoice could not be cleared: \r\n %s ") % clearance_data, 'blocking_level': 'error'}
        elif mode == 'reporting' and clearance_data.get('reportingStatus', '') != 'REPORTED':
            return {'error': _("Invoice could not be reported: \r\n %s ") % clearance_data, 'blocking_level': 'error'}
        return clearance_data

    # ====== UBL Document Rendering & Submission =======

    def _l10n_sa_postprocess_zatca_template(self, xml_content):
        """
            Post-process xml content generated according to the ZATCA UBL specifications. Specifically, this entails:
                -   Force the xmlns:ext namespace on the root element (Invoice). This is required, since, by default
                    the generated UBL file does not have any ext namespaced element, so the namespace is removed
                    since it is unused.
        """

        # Append UBLExtensions to the XML content
        ubl_extensions = etree.fromstring(self.env['ir.qweb']._render('l10n_sa_edi.export_sa_zatca_ubl_extensions'))
        root = etree.fromstring(xml_content)
        root.insert(0, ubl_extensions)

        # Force xmlns:ext namespace on UBl file
        ns_map = {'ext': 'urn:oasis:names:specification:ubl:schema:xsd:CommonExtensionComponents-2'}
        etree.cleanup_namespaces(root, top_nsmap=ns_map, keep_ns_prefixes=['ext'])

        return etree.tostring(root, with_tail=False).decode()

    def _l10n_sa_generate_zatca_template(self, invoice):
        """
            Render the ZATCA UBL file
        """
        xml_content, errors = self.env['account.edi.xml.ubl_21.zatca']._export_invoice(invoice)
        if errors:
            return {
                'error': _("Could not generate Invoice UBL content: %s") % ", \n".join(errors),
                'blocking_level': 'error'
            }
        return self._l10n_sa_postprocess_zatca_template(xml_content)

    def _l10n_sa_submit_einvoice(self, invoice, signed_xml, PCSID_data):
        """
            Submit a generated Invoice UBL file by making calls to the following APIs:
                -   A. Clearance API: Submit a standard Invoice to ZATCA for validation, returns signed UBL
                -   B. Reporting API: Submit a simplified Invoice to ZATCA for validation
        """
        clearance_data = invoice.journal_id._l10n_sa_api_clearance(invoice, signed_xml.decode(), PCSID_data)
        if clearance_data.get('json_errors'):
            error = clearance_data['json_errors']
            error_msg = ''
            status_code = error.get('status_code')
            if status_code:
                error_msg = Markup("<b>[%s] </b>") % status_code

            is_warning = True
            validation_results = error.get('validationResults', {})
            for err in validation_results.get('warningMessages', []):
                error_msg += Markup('<b>%s</b> : %s <br/>') % (err['code'], err['message'])
            for err in validation_results.get('errorMessages', []):
                is_warning = False
                error_msg += Markup('<b>%s</b> : %s <br/>') % (err['code'], err['message'])
            return {
                'error': error_msg,
                'rejected': not is_warning,
                'response': signed_xml.decode(),
                'blocking_level': 'warning' if is_warning else 'error',
                'status_code': status_code,
            }
        if not clearance_data.get('error'):
            return self._l10n_sa_assert_clearance_status(invoice, clearance_data)
        return clearance_data

    def _l10n_sa_postprocess_einvoice_submission(self, invoice, signed_xml, clearance_data):
        """
            Once an invoice has been successfully submitted, it is returned as a Cleared invoice, on which data
            from ZATCA was applied. To be overridden to account for other cases, such as Reporting.
        """
        if invoice._l10n_sa_is_simplified():
            # if invoice is B2C, it is a SIMPLIFIED invoice, and thus it is only reported and returns
            # no signed invoice. In this case, we just return the original content
            return signed_xml.decode()
        return b64decode(clearance_data['clearedInvoice']).decode()

    def _l10n_sa_apply_qr_code(self, invoice, xml_content):
        """
            Apply QR code on Invoice UBL content
        """
        root = etree.fromstring(xml_content)
        qr_code = invoice.l10n_sa_qr_code_str
        qr_node = root.xpath('//*[local-name()="ID"][text()="QR"]/following-sibling::*/*')[0]
        qr_node.text = qr_code
        return etree.tostring(root, with_tail=False)

    def _l10n_sa_get_signed_xml(self, invoice, unsigned_xml, x509_cert):
        """
            Helper method to sign the provided XML, apply the QR code in the case if Simplified invoices (B2C), then
            return the signed XML
        """
        signed_xml = self._l10n_sa_sign_xml(unsigned_xml, x509_cert, invoice.l10n_sa_invoice_signature)
        if invoice._l10n_sa_is_simplified():
            # Applying with_prefetch() to set the _prefetch_ids = _ids,
            # preventing premature QR code computation for other invoices.
            invoice = invoice.with_prefetch()
            return self._l10n_sa_apply_qr_code(invoice, signed_xml)
        return signed_xml

    def _l10n_sa_export_zatca_invoice(self, invoice, xml_content=None):
        """
            Generate a ZATCA compliant UBL file, make API calls to authenticate, sign and include QR Code and
            Cryptographic Stamp, then create an attachment with the final contents of the UBL file
        """
        self.ensure_one()

        # Prepare UBL invoice values and render XML file
        unsigned_xml = xml_content or self._l10n_sa_generate_zatca_template(invoice)

        # Load PCISD data and X509 certificate
        try:
            PCSID_data = invoice.journal_id._l10n_sa_api_get_pcsid()
        except UserError as e:
            return ({
                'error': _("Could not generate PCSID values: \n") + e.args[0],
                'blocking_level': 'error',
                'response': unsigned_xml
            }, unsigned_xml)
        x509_cert = PCSID_data['binarySecurityToken']

        # Apply Signature/QR code on the generated XML document
        try:
            signed_xml = self._l10n_sa_get_signed_xml(invoice, unsigned_xml, x509_cert)
        except UserError as e:
            return ({
                'error': _("Could not generate signed XML values: \n") + e.args[0],
                'blocking_level': 'error',
                'response': unsigned_xml
            }, unsigned_xml)

        # Once the XML content has been generated and signed, we submit it to ZATCA
        return self._l10n_sa_submit_einvoice(invoice, signed_xml, PCSID_data), signed_xml

    def _l10n_sa_check_partner_missing_info(self, partner_id, fields_to_check):
        """
            Helper function to check if ZATCA mandated partner fields are missing for a specified partner record
        """
        missing = []
        for field in fields_to_check:
            field_value = partner_id[field[0]]
            if not field_value or (len(field) == 3 and not field[2](partner_id, field_value)):
                missing.append(field[1])
        return missing

    def _l10n_sa_check_seller_missing_info(self, invoice):
        """
            Helper function to check if ZATCA mandated partner fields are missing for the seller
        """
        partner_id = invoice.company_id.partner_id.commercial_partner_id
        fields_to_check = [
            ('l10n_sa_edi_building_number', _('Building Number for the Buyer is required on Standard Invoices')),
            ('street2', _('Neighborhood for the Seller is required on Standard Invoices')),
            ('l10n_sa_additional_identification_scheme',
             _('Additional Identification Scheme is required for the Seller, and must be one of CRN, MOM, MLS, SAG or OTH'),
             lambda p, v: v in ('CRN', 'MOM', 'MLS', 'SAG', 'OTH')
             ),
            ('vat',
             _('VAT is required when Identification Scheme is set to Tax Identification Number'),
             lambda p, v: p.l10n_sa_additional_identification_scheme != 'TIN'
             ),
            ('state_id', _('State / Country subdivision'))
        ]
        return self._l10n_sa_check_partner_missing_info(partner_id, fields_to_check)

    def _l10n_sa_check_buyer_missing_info(self, invoice):
        """
            Helper function to check if ZATCA mandated partner fields are missing for the buyer
        """
        fields_to_check = []
        if any(tax.l10n_sa_exemption_reason_code in ('VATEX-SA-HEA', 'VATEX-SA-EDU') for tax in
               invoice.invoice_line_ids.filtered(
                   lambda line: line.display_type == 'product').tax_ids):
            fields_to_check += [
                ('l10n_sa_additional_identification_scheme',
                 _('Additional Identification Scheme is required for the Buyer if tax exemption reason is either '
                   'VATEX-SA-HEA or VATEX-SA-EDU, and its value must be NAT'), lambda p, v: v == 'NAT'),
                ('l10n_sa_additional_identification_number',
                 _('Additional Identification Number is required for commercial partners'),
                 lambda p, v: p.l10n_sa_additional_identification_scheme != 'TIN'
                 ),
            ]
        elif invoice.commercial_partner_id.l10n_sa_additional_identification_scheme == 'TIN':
            fields_to_check += [
                ('vat', _('VAT is required when Identification Scheme is set to Tax Identification Number'))
            ]
        if not invoice._l10n_sa_is_simplified() and invoice.partner_id.country_id.code == 'SA':
            # If the invoice is a non-foreign, Standard (B2B), the Building Number and Neighborhood are required
            fields_to_check += [
                ('l10n_sa_edi_building_number', _('Building Number for the Buyer is required on Standard Invoices')),
                ('street2', _('Neighborhood for the Buyer is required on Standard Invoices')),
            ]
        return self._l10n_sa_check_partner_missing_info(invoice.commercial_partner_id, fields_to_check)

    def _l10n_sa_post_zatca_edi(self, invoice):  # no batch ensure that there is only one invoice
        """
            Post invoice to ZATCA and return a dict of invoices and their success/attachment
        """

        # Chain integrity check: chain head must have been REALLY posted, and did not time out
        # When a submission times out, we reset the chain index of the invoice to False, so it has to be submitted again
        # According to ZATCA, if we end up submitting the same invoice more than once, they will directly reach out
        # to the taxpayer for clarifications
        chain_head = invoice.journal_id._l10n_sa_get_last_posted_invoice()
        if chain_head and chain_head != invoice and not chain_head._l10n_sa_is_in_chain():
            return {invoice: {
                'error': f"ZATCA: Cannot post invoice while chain head ({chain_head.name}) has not been posted",
                'blocking_level': 'error',
                'response': None,
            }}

        xml_content = None
        if not invoice.l10n_sa_chain_index:
            # If the Invoice doesn't have a chain index, it means it either has not been submitted before,
            # or it was submitted and rejected. Either way, we need to assign it a new Chain Index and regenerate
            # the data that depends on it before submitting (UUID, XML content, signature)
            invoice.l10n_sa_chain_index = invoice.journal_id._l10n_sa_edi_get_next_chain_index()
            xml_content = invoice._l10n_sa_generate_unsigned_data()

        # Generate Invoice name for attachment
        attachment_name = self.env['account.edi.xml.ubl_21.zatca']._export_invoice_filename(invoice)

        # Generate XML, sign it, then submit it to ZATCA
        response_data, submitted_xml = self._l10n_sa_export_zatca_invoice(invoice, xml_content)

        # Check for submission errors
        if response_data.get('error'):

            # If the request was rejected, we save the signed xml content as an attachment
            if response_data.get('rejected'):
                invoice._l10n_sa_log_results(submitted_xml, response_data, error=True)

            # If the request returned an exception (Timeout, ValueError... etc.) it means we're not sure if the
            # invoice was successfully cleared/reported, and thus we keep the Index Chain.
            # Else, we recalculate the submission Index (ICV), UUID, XML content and Signature
            if not response_data.get('excepted'):
                invoice.l10n_sa_chain_index = False

            return {
                invoice: {
                    **response_data,
                    'response': submitted_xml
                }
            }

        # Once submission is done with no errors, check submission status
        cleared_xml = self._l10n_sa_postprocess_einvoice_submission(invoice, submitted_xml, response_data)

        # Save the submitted/returned invoice XML content once the submission has been completed successfully
        invoice._l10n_sa_log_results(cleared_xml.encode(), response_data)
        return {
            invoice: {
                'success': True,
                'response': cleared_xml,
                'message': '',
                'attachment': self.env['ir.attachment'].create({
                    'name': attachment_name,
                    'raw': cleared_xml.encode(),
                    'res_model': 'account.move',
                    'res_id': invoice.id,
                    'mimetype': 'application/xml'
                })
            }
        }

    # ====== EDI Format Overrides =======

    def _is_required_for_invoice(self, invoice):
        """
            Override to add ZATCA edi checks on required invoices
        """
        self.ensure_one()
        if self.code != 'sa_zatca':
            return super()._is_required_for_invoice(invoice)

        return invoice.is_sale_document() and invoice.country_code == 'SA'

    def _check_move_configuration(self, invoice):
        """
            Override to add ZATCA compliance checks on the Invoice
        """

        def _set_missing_partner_fields(missing_fields, name):
            return _("- Please, set the following fields on the %s: %s") % (name, ', '.join(missing_fields))

        journal = invoice.journal_id
        company = invoice.company_id

        errors = super()._check_move_configuration(invoice)
        if self.code != 'sa_zatca' or company.country_id.code != 'SA':
            return errors

        if invoice.commercial_partner_id == invoice.company_id.partner_id.commercial_partner_id:
            errors.append(_("- You cannot post invoices where the Seller is the Buyer"))

        if not all(line.tax_ids for line in invoice.invoice_line_ids.filtered(lambda line: line.display_type == 'product')):
            errors.append(_("- Invoice lines should have at least one Tax applied."))

        if not journal._l10n_sa_ready_to_submit_einvoices():
            errors.append(
                _("- Finish the Onboarding procees for journal %s by requesting the CSIDs and completing the checks.") % journal.name)

        if not company._l10n_sa_check_organization_unit():
            errors.append(
                _("- The company VAT identification must contain 15 digits, with the first and last digits being '3' as per the BR-KSA-39 and BR-KSA-40 of ZATCA KSA business rule."))
        if not company.sudo().l10n_sa_private_key:
            errors.append(
                _("- No Private Key was generated for company %s. A Private Key is mandatory in order to generate Certificate Signing Requests (CSR).") % company.name)
        if not journal.l10n_sa_serial_number:
            errors.append(
                _("- No Serial Number was assigned for journal %s. A Serial Number is mandatory in order to generate Certificate Signing Requests (CSR).") % journal.name)

        supplier_missing_info = self._l10n_sa_check_seller_missing_info(invoice)
        customer_missing_info = self._l10n_sa_check_buyer_missing_info(invoice)

        if supplier_missing_info:
            errors.append(_set_missing_partner_fields(supplier_missing_info, _("Supplier")))
        if customer_missing_info:
            errors.append(_set_missing_partner_fields(customer_missing_info, _("Customer")))
        if invoice.invoice_date > fields.Date.context_today(self.with_context(tz='Asia/Riyadh')):
            errors.append(_("- Please, make sure the invoice date is set to either the same as or before Today."))
        if invoice.move_type in ('in_refund', 'out_refund') and not invoice._l10n_sa_check_refund_reason():
            errors.append(
                _("- Please, make sure either the Reversed Entry or the Reversal Reason are specified when confirming a Credit/Debit note"))
        return errors

    def _needs_web_services(self):
        """
            Override to add a check on edi document format code
        """
        self.ensure_one()
        return self.code == 'sa_zatca' or super()._needs_web_services()

    def _is_compatible_with_journal(self, journal):
        """
            Override to add a check on journal type & country code (SA)
        """
        self.ensure_one()
        if self.code != 'sa_zatca':
            return super()._is_compatible_with_journal(journal)
        return journal.type == 'sale' and journal.country_code == 'SA'

    def _l10n_sa_get_invoice_content_edi(self, invoice):
        """
            Return contents of the submitted UBL file or generate it if the invoice has not been submitted yet
        """
        doc = invoice.edi_document_ids.filtered(lambda d: d.edi_format_id.code == 'sa_zatca' and d.state == 'sent')
        return doc.attachment_id.raw or self._l10n_sa_generate_zatca_template(invoice).encode()

    def _get_move_applicability(self, move):
        # EXTENDS account_edi
        self.ensure_one()
        if self.code != 'sa_zatca' or move.country_code != 'SA' or move.move_type not in ('out_invoice', 'out_refund'):
            return super()._get_move_applicability(move)

        return {
            'post': self._l10n_sa_post_zatca_edi,
            'edi_content': self._l10n_sa_get_invoice_content_edi,
        }

    def _prepare_invoice_report(self, pdf_writer, edi_document):
        """
        Prepare invoice report to be printed.
        :param pdf_writer: The pdf writer with the invoice pdf content loaded.
        :param edi_document: The edi document to be added to the pdf file.
        """
        self.ensure_one()
        super()._prepare_invoice_report(pdf_writer, edi_document)
        if self.code != 'sa_zatca' or edi_document.move_id.country_code != 'SA':
            return

        attachment = edi_document.sudo().attachment_id
        if not attachment or not attachment.datas:
            _logger.warning(f"No attachment found for invoice {edi_document.move_id.name}")
            return

        xml_content = attachment.raw
        file_name = attachment.name

        pdf_writer.addAttachment(file_name, xml_content, subtype='text/xml')
        if not pdf_writer.is_pdfa:
            try:
                pdf_writer.convert_to_pdfa()
            except Exception as e:
                _logger.exception("Error while converting to PDF/A: %s", e)
            content = self.env['ir.qweb']._render(
                'account_edi_ubl_cii.account_invoice_pdfa_3_facturx_metadata',
                {
                    'title': edi_document.move_id.name,
                    'date': fields.Date.context_today(self),
                },
            )
            pdf_writer.add_file_metadata(content.encode())

```

## File: models\account_edi_xml_ubl_21_zatca.py

```python
# -*- coding: utf-8 -*-
from hashlib import sha256
from base64 import b64encode
from lxml import etree
from odoo import models, fields
from odoo.modules.module import get_module_resource
import re

TAX_EXEMPTION_CODES = ['VATEX-SA-29', 'VATEX-SA-29-7', 'VATEX-SA-30']
TAX_ZERO_RATE_CODES = ['VATEX-SA-32', 'VATEX-SA-33', 'VATEX-SA-34-1', 'VATEX-SA-34-2', 'VATEX-SA-34-3', 'VATEX-SA-34-4',
                       'VATEX-SA-34-5', 'VATEX-SA-35', 'VATEX-SA-36', 'VATEX-SA-EDU', 'VATEX-SA-HEA']

PAYMENT_MEANS_CODE = {
    'bank': 42,
    'card': 48,
    'cash': 10,
    'transfer': 30,
    'unknown': 1
}


class AccountEdiXmlUBL21Zatca(models.AbstractModel):
    _name = "account.edi.xml.ubl_21.zatca"
    _inherit = 'account.edi.xml.ubl_21'
    _description = "UBL 2.1 (ZATCA)"

    def _l10n_sa_get_namespaces(self):
        """
            Namespaces used in the final UBL declaration, required to canonalize the finalized XML document of the Invoice
        """
        return {
            'cac': 'urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2',
            'cbc': 'urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2',
            'ext': 'urn:oasis:names:specification:ubl:schema:xsd:CommonExtensionComponents-2',
            'sig': 'urn:oasis:names:specification:ubl:schema:xsd:CommonSignatureComponents-2',
            'sac': 'urn:oasis:names:specification:ubl:schema:xsd:SignatureAggregateComponents-2',
            'sbc': 'urn:oasis:names:specification:ubl:schema:xsd:SignatureBasicComponents-2',
            'ds': 'http://www.w3.org/2000/09/xmldsig#',
            'xades': 'http://uri.etsi.org/01903/v1.3.2#'
        }

    def _l10n_sa_generate_invoice_xml_sha(self, xml_content):
        """
            Transform, canonicalize then hash the invoice xml content using the SHA256 algorithm,
            then return the hashed content
        """

        def _canonicalize_xml(content):
            """
                Canonicalize XML content using the c14n method. The specs mention using the c14n11 canonicalization,
                which is simply calling etree.tostring and setting the method argument to 'c14n'. There are minor
                differences between c14n11 and c14n canonicalization algorithms, but for the purpose of ZATCA signing,
                c14n is enough
            """
            return etree.tostring(content, method="c14n", exclusive=False, with_comments=False,
                                  inclusive_ns_prefixes=self._l10n_sa_get_namespaces())

        def _transform_and_canonicalize_xml(content):
            """ Transform XML content to remove certain elements and signatures using an XSL template """
            invoice_xsl = etree.parse(get_module_resource('l10n_sa_edi', 'data', 'pre-hash_invoice.xsl'))
            transform = etree.XSLT(invoice_xsl)
            return _canonicalize_xml(transform(content))

        root = etree.fromstring(xml_content)
        # Transform & canonicalize the XML content
        transformed_xml = _transform_and_canonicalize_xml(root)
        # Get the SHA256 hashed value of the XML content
        return sha256(transformed_xml)

    def _l10n_sa_generate_invoice_xml_hash(self, xml_content, mode='hexdigest'):
        """
            Generate the b64 encoded sha256 hash of a given xml string:
                - First: Transform the xml content using a pre-hash_invoice.xsl file
                - Second: Canonicalize the transformed xml content using the c14n method
                - Third: hash the canonicalized content using the sha256 algorithm then encode it into b64 format
        """
        xml_sha = self._l10n_sa_generate_invoice_xml_sha(xml_content)
        if mode == 'hexdigest':
            xml_hash = xml_sha.hexdigest().encode()
        elif mode == 'digest':
            xml_hash = xml_sha.digest()
        return b64encode(xml_hash)

    def _l10n_sa_get_previous_invoice_hash(self, invoice):
        """ Function that returns the Base 64 encoded SHA256 hash of the previously submitted invoice """
        if invoice.company_id.l10n_sa_api_mode == 'sandbox' or not invoice.journal_id.l10n_sa_latest_submission_hash:
            # If no invoice, or if using Sandbox, return the b64 encoded SHA256 value of the '0' character
            return "NWZlY2ViNjZmZmM4NmYzOGQ5NTI3ODZjNmQ2OTZjNzljMmRiYzIzOWRkNGU5MWI0NjcyOWQ3M2EyN2ZiNTdlOQ=="
        return invoice.journal_id.l10n_sa_latest_submission_hash

    def _get_delivery_vals_list(self, invoice):
        """ Override to include/update values specific to ZATCA's UBL 2.1 specs """
        shipping_address = invoice.partner_shipping_id
        return [{'actual_delivery_date': invoice.l10n_sa_delivery_date,
                 'delivery_address_vals': self._get_partner_address_vals(shipping_address) if shipping_address else {},}]

    def _get_partner_contact_vals(self, partner):
        res = super()._get_partner_contact_vals(partner)
        if res.get('telephone'):
            res['telephone'] = re.sub(r"[^+\d]", '', res['telephone'])
        return res

    def _get_partner_party_identification_vals_list(self, partner):
        """ Override to include/update values specific to ZATCA's UBL 2.1 specs """
        return [{
            'id_attrs': {'schemeID': partner.l10n_sa_additional_identification_scheme},
            'id': (
                partner.l10n_sa_additional_identification_number
                if partner.l10n_sa_additional_identification_scheme != 'TIN' and partner.country_code == 'SA'
                else partner.vat
            ),
        }]

    def _get_partner_party_legal_entity_vals_list(self, partner):
        # EXTEND 'account.edi.xml.ubl_20'
        partners_party_legal = super()._get_partner_party_legal_entity_vals_list(partner)
        for partner_party_legal in partners_party_legal:
            if partner_party_legal['commercial_partner'].country_code != 'SA':
                partner_party_legal['company_id'] = False

        return partners_party_legal

    def _l10n_sa_get_payment_means_code(self, invoice):
        """ Return payment means code to be used to set the value on the XML file """
        return 'unknown'

    def _get_invoice_payment_means_vals_list(self, invoice):
        """ Override to include/update values specific to ZATCA's UBL 2.1 specs """
        res = super()._get_invoice_payment_means_vals_list(invoice)
        res[0]['payment_means_code'] = PAYMENT_MEANS_CODE.get(self._l10n_sa_get_payment_means_code(invoice), PAYMENT_MEANS_CODE['unknown'])
        res[0]['payment_means_code_attrs'] = {'listID': 'UN/ECE 4461'}
        res[0]['adjustment_reason'] = invoice.ref
        return res

    def _get_partner_address_vals(self, partner):
        """ Override to include/update values specific to ZATCA's UBL 2.1 specs """
        return {
            **super()._get_partner_address_vals(partner),
            'building_number': partner.l10n_sa_edi_building_number,
            'neighborhood': partner.street2,
            'plot_identification': partner.l10n_sa_edi_plot_identification,
        }

    def _export_invoice_filename(self, invoice):
        """
            Generate the name of the invoice XML file according to ZATCA business rules:
            Seller Vat Number (BT-31), Date (BT-2), Time (KSA-25), Invoice Number (BT-1)
        """
        vat = invoice.company_id.partner_id.commercial_partner_id.vat
        invoice_number = re.sub(r'[^a-zA-Z0-9 -]+', '-', invoice.name)
        invoice_date = fields.Datetime.context_timestamp(self.with_context(tz='Asia/Riyadh'), invoice.l10n_sa_confirmation_datetime)
        file_name = f"{vat}_{invoice_date.strftime('%Y%m%dT%H%M%S')}_{invoice_number}"
        file_format = self.env.context.get('l10n_sa_file_format', 'xml')
        if file_format:
            file_name = f'{file_name}.{file_format}'
        return file_name

    def _l10n_sa_get_invoice_transaction_code(self, invoice):
        """
            Returns the transaction code string to be inserted in the UBL file follows the following format:
                - NNPNESB, in compliance with KSA Business Rule KSA-2, where:
                    - NN (positions 1 and 2) = invoice subtype:
                        - 01 for tax invoice
                        - 02 for simplified tax invoice
                    - E (position 5) = Exports invoice transaction, 0 for false, 1 for true
        """
        return '0%s00%s00' % (
            '2' if invoice._l10n_sa_is_simplified() else '1',
            '1' if invoice.commercial_partner_id.country_id != invoice.company_id.country_id and not invoice._l10n_sa_is_simplified() else '0'
        )

    def _l10n_sa_get_invoice_type(self, invoice):
        """
            Returns the invoice type string to be inserted in the UBL file
                - 383: Debit Note
                - 381: Credit Note
                - 388: Invoice
        """
        return (
            383 if invoice.debit_origin_id else
            381 if invoice.move_type == 'out_refund' else
            386 if invoice._is_downpayment() else 388
        )

    def _l10n_sa_get_billing_reference_vals(self, invoice):
        """ Get the billing reference vals required to render the BillingReference for credit/debit notes """
        if self._l10n_sa_get_invoice_type(invoice) != 388:
            return {
                'id': (invoice.reversed_entry_id.name or invoice.ref) if invoice.move_type == 'out_refund' else invoice.debit_origin_id.name,
                'issue_date': None,
            }
        return {}

    def _get_partner_party_tax_scheme_vals_list(self, partner, role):
        """
            Override to return an empty list if the partner is a customer and their country is not KSA.
            This is according to KSA Business Rule BR-KSA-46 which states that in the case of Export Invoices,
            the buyer VAT registration number or buyer group VAT registration number must not exist in the Invoice
        """
        if role != 'customer' or partner.country_id.code == 'SA':
            return super()._get_partner_party_tax_scheme_vals_list(partner, role)
        return []

    def _apply_invoice_tax_filter(self, base_line, tax_values):
        """ Override to filter out withholding tax """
        tax_id = self.env['account.tax'].browse(tax_values['id'])
        res = not tax_id.l10n_sa_is_retention
        # If the move that is being sent is not a down payment invoice, and the sale module is installed
        # we need to make sure the line is neither retention, nor a down payment line
        if not base_line['record'].move_id._is_downpayment():
            return not tax_id.l10n_sa_is_retention and not base_line['record']._get_downpayment_lines()
        return res

    def _apply_invoice_line_filter(self, invoice_line):
        """ Override to filter out down payment lines """
        if not invoice_line.move_id._is_downpayment():
            return not invoice_line._get_downpayment_lines()
        return True

    def _l10n_sa_get_prepaid_amount(self, invoice, vals):
        """ Calculate the down-payment amount according to ZATCA rules """
        downpayment_lines = False if invoice._is_downpayment() else invoice.line_ids.filtered(lambda l: l._get_downpayment_lines())
        if downpayment_lines:
            tax_vals = invoice._prepare_edi_tax_details(filter_to_apply=lambda l, t: not self.env['account.tax'].browse(t['id']).l10n_sa_is_retention)
            base_amount = abs(sum(tax_vals['tax_details_per_record'][l]['base_amount_currency'] for l in downpayment_lines))
            tax_amount = abs(sum(tax_vals['tax_details_per_record'][l]['tax_amount_currency'] for l in downpayment_lines))
            return {
                'total_amount': base_amount + tax_amount,
                'base_amount': base_amount,
                'tax_amount': tax_amount
            }

    def _l10n_sa_get_monetary_vals(self, invoice, vals):
        """ Calculate the invoice monteray amount values, including prepaid amounts (down payment) """
        # We use base_amount_currency + tax_amount_currency instead of amount_total because we do not want to include
        # withholding tax amounts in our calculations
        total_amount = abs(vals['taxes_vals']['base_amount_currency'] + vals['taxes_vals']['tax_amount_currency'])
        line_extension_amount = vals['vals']['legal_monetary_total_vals']['line_extension_amount']
        tax_inclusive_amount = total_amount
        tax_exclusive_amount = abs(vals['taxes_vals']['base_amount_currency'])
        prepaid_amount = 0
        payable_amount = total_amount
        # - When we calculate the tax values, we filter out taxes and invoice lines linked to downpayments.
        #   As such, when we calculate the TaxInclusiveAmount, it already accounts for the tax amount of the downpayment
        #   Same goes for the TaxExclusiveAmount, and we do not need to add the Tax amount of the downpayment
        # - The payable amount does not account for the tax amount of the downpayment, so we add it
        downpayment_vals = self._l10n_sa_get_prepaid_amount(invoice, vals)
        allowance_charge_vals = vals['vals']['allowance_charge_vals']
        allowance_total_amount = sum(a['amount'] for a in allowance_charge_vals if a['charge_indicator'] == 'false')
        if downpayment_vals:
            # - BR-KSA-80: To calculate payable amount, we deduct prepaid amount from total tax inclusive amount
            prepaid_amount = downpayment_vals['total_amount']
            payable_amount = tax_inclusive_amount - prepaid_amount
        return {
            'line_extension_amount': line_extension_amount - allowance_total_amount,
            'tax_inclusive_amount': tax_inclusive_amount,
            'tax_exclusive_amount': tax_exclusive_amount,
            'prepaid_amount': prepaid_amount,
            'payable_amount': payable_amount,
            'allowance_total_amount': allowance_total_amount
        }

    def _get_tax_category_list(self, invoice, taxes):
        """ Override to filter out withholding taxes """
        non_retention_taxes = taxes.filtered(lambda t: not t.l10n_sa_is_retention)
        return super()._get_tax_category_list(invoice, non_retention_taxes)

    def _get_document_allowance_charge_vals_list(self, invoice):
        """
        Charge Reasons & Codes (As per ZATCA):
        https://unece.org/fileadmin/DAM/trade/untdid/d16b/tred/tred5189.htm
        As far as ZATCA is concerned, we calculate Allowance/Charge vals for global discounts as
        a document level allowance, and we do not include any other charges or allowances
        """
        res = super()._get_document_allowance_charge_vals_list(invoice)
        for line in invoice.invoice_line_ids.filtered(lambda l: l._is_global_discount_line()):
            taxes = line.tax_ids.flatten_taxes_hierarchy().filtered(lambda t: t.amount_type != 'fixed')
            res.append({
                'charge_indicator': 'false',
                'allowance_charge_reason_code': "95",
                'allowance_charge_reason': "Discount",
                'amount': abs(line.price_subtotal),
                'currency_dp': 2,
                'currency_name': invoice.currency_id.name,
                'tax_category_vals': [{
                    'id': tax['id'],
                    'percent': tax['percent'],
                    'tax_scheme_id': 'VAT',
                } for tax in self._get_tax_category_list(line.move_id, taxes)],
            })
        return res

    def _export_invoice_vals(self, invoice):
        """ Override to include/update values specific to ZATCA's UBL 2.1 specs """
        vals = super()._export_invoice_vals(invoice)

        vals.update({
            'main_template': 'account_edi_ubl_cii.ubl_20_Invoice',
            'InvoiceType_template': 'l10n_sa_edi.ubl_21_InvoiceType_zatca',
            'InvoiceLineType_template': 'l10n_sa_edi.ubl_21_InvoiceLineType_zatca',
            'AddressType_template': 'l10n_sa_edi.ubl_21_AddressType_zatca',
            'PartyType_template': 'l10n_sa_edi.ubl_21_PartyType_zatca',
            'TaxTotalType_template': 'l10n_sa_edi.ubl_21_TaxTotalType_zatca',
            'PaymentMeansType_template': 'l10n_sa_edi.ubl_21_PaymentMeansType_zatca',
        })

        vals['vals'].update({
            'profile_id': 'reporting:1.0',
            'invoice_type_code_attrs': {'name': self._l10n_sa_get_invoice_transaction_code(invoice)},
            'invoice_type_code': self._l10n_sa_get_invoice_type(invoice),
            'issue_date': fields.Datetime.context_timestamp(self.with_context(tz='Asia/Riyadh'),
                                                            invoice.l10n_sa_confirmation_datetime),
            'previous_invoice_hash': self._l10n_sa_get_previous_invoice_hash(invoice),
            'billing_reference_vals': self._l10n_sa_get_billing_reference_vals(invoice),
            'tax_total_vals': self._l10n_sa_get_additional_tax_total_vals(invoice, vals),
            # Due date is not required for ZATCA UBL 2.1
            'due_date': None,
        })

        vals['vals']['legal_monetary_total_vals'].update(self._l10n_sa_get_monetary_vals(invoice, vals))
        self._l10n_sa_postprocess_line_vals(vals)

        return vals

    def _l10n_sa_postprocess_line_vals(self, vals):
        """
            Postprocess vals to remove negative line amounts, as those will be used to compute
            document level allowances (global discounts)
        """
        final_line_vals = []
        for line_vals in vals['vals']['invoice_line_vals']:
            if line_vals['price_vals']['price_amount'] >= 0:
                final_line_vals.append(line_vals)
        vals['vals']['invoice_line_vals'] = final_line_vals

    def _l10n_sa_get_additional_tax_total_vals(self, invoice, vals):
        """
            For ZATCA, an additional TaxTotal element needs to be included in the UBL file
            (Only for the Invoice, not the lines)

            If the invoice is in a different currency from the one set on the company (SAR), then the additional
            TaxAmount element needs to hold the tax amount converted to the company's currency.

            Business Rules: BT-110 & BT-111
        """
        curr_amount = abs(vals['taxes_vals']['tax_amount_currency'])
        if invoice.currency_id != invoice.company_currency_id:
            curr_amount = abs(vals['taxes_vals']['tax_amount'])
        return vals['vals']['tax_total_vals'] + [{
            'currency': invoice.company_currency_id,
            'currency_dp': invoice.company_currency_id.decimal_places,
            'tax_amount': curr_amount,
        }]

    def _get_invoice_line_item_vals(self, line, taxes_vals):
        """ Override to include/update values specific to ZATCA's UBL 2.1 specs """
        vals = super()._get_invoice_line_item_vals(line, taxes_vals)
        vals['sellers_item_identification_vals'] = {'id': line.product_id.code or line.product_id.default_code}
        return vals

    def _l10n_sa_get_line_prepayment_vals(self, line, taxes_vals):
        """
            If an invoice line is linked to a down payment invoice, we need to return the proper values
            to be included in the UBL
        """
        if not line.move_id._is_downpayment() and line.sale_line_ids and all(sale_line.is_downpayment for sale_line in line.sale_line_ids):
            prepayment_move_id = line.sale_line_ids.invoice_lines.move_id.filtered(lambda m: m._is_downpayment())
            return {
                'prepayment_id': prepayment_move_id.name,
                'issue_date': fields.Datetime.context_timestamp(self.with_context(tz='Asia/Riyadh'),
                                                                prepayment_move_id.l10n_sa_confirmation_datetime),
                'document_type_code': 386
            }
        return {}

    def _get_invoice_line_vals(self, line, taxes_vals):
        """ Override to include/update values specific to ZATCA's UBL 2.1 specs """

        def grouping_key_generator(base_line, tax_values):
            tax = tax_values['tax_repartition_line'].tax_id
            tax_category_vals = self._get_tax_category_list(line.move_id, tax)[0]
            grouping_key = {
                'tax_category_id': tax_category_vals['id'],
                'tax_category_percent': tax_category_vals['percent'],
                '_tax_category_vals_': tax_category_vals,
                'tax_amount_type': tax.amount_type,
            }
            if tax.amount_type == 'fixed':
                grouping_key['tax_name'] = tax.name
            return grouping_key

        if not line.move_id._is_downpayment() and line._get_downpayment_lines():
            # When we initially calculate the taxes_vals, we filter out the down payment lines, which means we have no
            # values to set in the TaxableAmount and TaxAmount nodes on the InvoiceLine for the down payment.
            # This means ZATCA will return a warning message for the BR-KSA-80 rule since it cannot calculate the
            # TaxableAmount and the TaxAmount nodes correctly. To avoid this, we re-caclculate the taxes_vals just before
            # we set the values for the down payment line, and we do not pass any filters to the _prepare_edi_tax_details
            # method
            line_taxes = line.move_id._prepare_edi_tax_details(grouping_key_generator=grouping_key_generator)
            taxes_vals = line_taxes['tax_details_per_record'][line]

        line_vals = super()._get_invoice_line_vals(line, taxes_vals)
        total_amount_sa = abs(taxes_vals['tax_amount_currency'] + taxes_vals['base_amount_currency'])
        extension_amount = abs(line_vals['line_extension_amount'])
        if not line.move_id._is_downpayment() and line._get_downpayment_lines():
            total_amount_sa = extension_amount = 0
            line_vals['price_vals']['price_amount'] = 0
            line_vals['tax_total_vals'][0]['tax_amount'] = 0
            line_vals['prepayment_vals'] = self._l10n_sa_get_line_prepayment_vals(line, taxes_vals)
        else:
            # - BR-KSA-80: only down-payment lines should have a tax subtotal breakdown, as that is
            # used during computation of prepaid amount as ZATCA sums up tax amount/taxable amount of all lines
            # irrespective of whether they are down-payment lines.
            line_vals['tax_total_vals'][0].pop('tax_subtotal_vals', None)
        line_vals['tax_total_vals'][0]['total_amount_sa'] = total_amount_sa
        line_vals['invoiced_quantity'] = abs(line_vals['invoiced_quantity'])
        line_vals['line_extension_amount'] = extension_amount

        return line_vals

    def _get_invoice_tax_totals_vals_list(self, invoice, taxes_vals):
        """
            Override to include/update values specific to ZATCA's UBL 2.1 specs.
            In this case, we make sure the tax amounts are always absolute (no negative values)
        """
        res = [{
            'currency': invoice.currency_id,
            'currency_dp': invoice.currency_id.decimal_places,
            'tax_amount': abs(taxes_vals['tax_amount_currency']),
            'tax_subtotal_vals': [{
                'currency': invoice.currency_id,
                'currency_dp': invoice.currency_id.decimal_places,
                'taxable_amount': abs(vals['base_amount_currency']),
                'tax_amount': abs(vals['tax_amount_currency']),
                'percent': vals['_tax_category_vals_']['percent'],
                'tax_category_vals': vals['_tax_category_vals_'],
            } for vals in taxes_vals['tax_details'].values()],
        }]
        return res

    def _get_tax_unece_codes(self, invoice, tax):
        """ Override to include/update values specific to ZATCA's UBL 2.1 specs """

        def _exemption_reason(code, reason):
            return {
                'tax_category_code': code,
                'tax_exemption_reason_code': reason or "VATEX-SA-OOS",
                'tax_exemption_reason': (
                    exemption_codes[reason].split(reason)[1].lstrip()
                    if reason else "Not subject to VAT"
                )
            }

        supplier = invoice.company_id.partner_id.commercial_partner_id
        if supplier.country_id.code == 'SA':
            if not tax or tax.amount == 0:
                exemption_codes = dict(tax._fields["l10n_sa_exemption_reason_code"]._description_selection(self.env))
                if tax.l10n_sa_exemption_reason_code in TAX_EXEMPTION_CODES:
                    return _exemption_reason('E', tax.l10n_sa_exemption_reason_code)
                elif tax.l10n_sa_exemption_reason_code in TAX_ZERO_RATE_CODES:
                    return _exemption_reason('Z', tax.l10n_sa_exemption_reason_code)
                else:
                    return _exemption_reason('O', tax.l10n_sa_exemption_reason_code)
            else:
                return {
                    'tax_category_code': 'S',
                    'tax_exemption_reason_code': None,
                    'tax_exemption_reason': None,
                }
        return super()._get_tax_unece_codes(invoice, tax)

    def _get_invoice_payment_terms_vals_list(self, invoice):
        """ Override to include/update values specific to ZATCA's UBL 2.1 specs """
        return []

```

## File: models\account_journal.py

```python
import json
import requests
from markupsafe import Markup
from lxml import etree
from datetime import datetime
from base64 import b64encode, b64decode
from odoo import models, fields, service, _, api
from odoo.exceptions import UserError
from odoo.modules.module import get_module_resource
from requests.exceptions import HTTPError, RequestException
from cryptography import x509
from cryptography.x509 import ObjectIdentifier, load_der_x509_certificate
from cryptography.x509.oid import NameOID
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.serialization import Encoding, load_pem_private_key
from urllib.parse import urljoin

ZATCA_API_URLS = {
    "sandbox": "https://gw-fatoora.zatca.gov.sa/e-invoicing/developer-portal/",
    "preprod": "https://gw-fatoora.zatca.gov.sa/e-invoicing/simulation/",
    "prod": "https://gw-fatoora.zatca.gov.sa/e-invoicing/core/",
    "apis": {
        "ccsid": "compliance",
        "pcsid": "production/csids",
        "compliance": "compliance/invoices",
        "reporting": "invoices/reporting/single",
        "clearance": "invoices/clearance/single",
    }
}

CERT_TEMPLATE_NAME = {
    'prod': b'\x0c\x12ZATCA-Code-Signing',
    'sandbox': b'\x13\x15PREZATCA-Code-Signing',
    'preprod': b'\x13\x15PREZATCA-Code-Signing',
}
# This SANDBOX_AUTH is only used for testing purposes, and is shared to all users of the sandbox environment
SANDBOX_AUTH = {
    'binarySecurityToken': "TUlJRDFEQ0NBM21nQXdJQkFnSVRid0FBZTNVQVlWVTM0SS8rNVFBQkFBQjdkVEFLQmdncWhrak9QUVFEQWpCak1SVXdFd1lLQ1pJbWlaUHlMR1FCR1JZRmJHOWpZV3d4RXpBUkJnb0praWFKay9Jc1pBRVpGZ05uYjNZeEZ6QVZCZ29Ka2lhSmsvSXNaQUVaRmdkbGVIUm5ZWHAwTVJ3d0dnWURWUVFERXhOVVUxcEZTVTVXVDBsRFJTMVRkV0pEUVMweE1CNFhEVEl5TURZeE1qRTNOREExTWxvWERUSTBNRFl4TVRFM05EQTFNbG93U1RFTE1Ba0dBMVVFQmhNQ1UwRXhEakFNQmdOVkJBb1RCV0ZuYVd4bE1SWXdGQVlEVlFRTEV3MW9ZWGxoSUhsaFoyaHRiM1Z5TVJJd0VBWURWUVFERXdreE1qY3VNQzR3TGpFd1ZqQVFCZ2NxaGtqT1BRSUJCZ1VyZ1FRQUNnTkNBQVRUQUs5bHJUVmtvOXJrcTZaWWNjOUhEUlpQNGI5UzR6QTRLbTdZWEorc25UVmhMa3pVMEhzbVNYOVVuOGpEaFJUT0hES2FmdDhDL3V1VVk5MzR2dU1ObzRJQ0p6Q0NBaU13Z1lnR0ExVWRFUVNCZ0RCK3BId3dlakViTUJrR0ExVUVCQXdTTVMxb1lYbGhmREl0TWpNMGZETXRNVEV5TVI4d0hRWUtDWkltaVpQeUxHUUJBUXdQTXpBd01EYzFOVGc0TnpBd01EQXpNUTB3Q3dZRFZRUU1EQVF4TVRBd01SRXdEd1lEVlFRYURBaGFZWFJqWVNBeE1qRVlNQllHQTFVRUR3d1BSbTl2WkNCQ2RYTnphVzVsYzNNek1CMEdBMVVkRGdRV0JCU2dtSVdENmJQZmJiS2ttVHdPSlJYdkliSDlIakFmQmdOVkhTTUVHREFXZ0JSMllJejdCcUNzWjFjMW5jK2FyS2NybVRXMUx6Qk9CZ05WSFI4RVJ6QkZNRU9nUWFBL2hqMW9kSFJ3T2k4dmRITjBZM0pzTG5waGRHTmhMbWR2ZGk1ellTOURaWEowUlc1eWIyeHNMMVJUV2tWSlRsWlBTVU5GTFZOMVlrTkJMVEV1WTNKc01JR3RCZ2dyQmdFRkJRY0JBUVNCb0RDQm5UQnVCZ2dyQmdFRkJRY3dBWVppYUhSMGNEb3ZMM1J6ZEdOeWJDNTZZWFJqWVM1bmIzWXVjMkV2UTJWeWRFVnVjbTlzYkM5VVUxcEZhVzUyYjJsalpWTkRRVEV1WlhoMFoyRjZkQzVuYjNZdWJHOWpZV3hmVkZOYVJVbE9WazlKUTBVdFUzVmlRMEV0TVNneEtTNWpjblF3S3dZSUt3WUJCUVVITUFHR0gyaDBkSEE2THk5MGMzUmpjbXd1ZW1GMFkyRXVaMjkyTG5OaEwyOWpjM0F3RGdZRFZSMFBBUUgvQkFRREFnZUFNQjBHQTFVZEpRUVdNQlFHQ0NzR0FRVUZCd01DQmdnckJnRUZCUWNEQXpBbkJna3JCZ0VFQVlJM0ZRb0VHakFZTUFvR0NDc0dBUVVGQndNQ01Bb0dDQ3NHQVFVRkJ3TURNQW9HQ0NxR1NNNDlCQU1DQTBrQU1FWUNJUUNWd0RNY3E2UE8rTWNtc0JYVXovdjFHZGhHcDdycVNhMkF4VEtTdjgzOElBSWhBT0JOREJ0OSszRFNsaWpvVmZ4enJkRGg1MjhXQzM3c21FZG9HV1ZyU3BHMQ==",
    'secret': "Xlj15LyMCgSC66ObnEO/qVPfhSbs3kDTjWnGheYhfSs="
}


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    """
        In order to clear/report an invoice through the ZATCA API, we need to onboard each journal by following
        three steps:

            STEP 1:
                Make a call to the Compliance CSID API '/compliance'.
                This will return three things:
                    -   X509 Compliance Cryptographic Stamp Identifier (CCSID/Certificate)
                    -   Password (Secret)
                    -   Compliance Request ID
            STEP 2:
                Make a call to the Compliance Checks API '/compliance/invoices', by passing the hashed xml content
                of the files available in the tests/compliance folder. This will check if the provided
                Standard/Simplified Invoices comply with UBL 2.1 standards in line with ZATCA specifications
            STEP 3:
                Make a call to the Production CSID API '/production/csids' including the Compliance Certificate,
                Password and Request ID from STEP 1.
                This will return three things:
                    -   X509 Production Certificate
                    -   Password (Secret)
                    -   Production Request ID
    """

    l10n_sa_csr = fields.Binary(attachment=True, copy=False, groups="base.group_system",
                                help="The Certificate Signing Request that is submitted to the Compliance API")
    l10n_sa_csr_errors = fields.Html("Onboarding Errors", copy=False)

    l10n_sa_compliance_csid_json = fields.Char("CCSID JSON", copy=False, groups="base.group_system",
                                               help="Compliance CSID data received from the Compliance CSID API "
                                                    "in dumped json format")
    l10n_sa_production_csid_json = fields.Char("PCSID JSON", copy=False, groups="base.group_system",
                                               help="Production CSID data received from the Production CSID API "
                                                    "in dumped json format")
    l10n_sa_production_csid_validity = fields.Datetime("PCSID Expiration", help="Production CSID expiration date",
                                                       compute="_l10n_sa_compute_production_csid_validity", store=True)
    l10n_sa_compliance_checks_passed = fields.Boolean("Compliance Checks Done", default=False, copy=False,
                                                      help="Specifies if the Compliance Checks have been completed successfully")

    l10n_sa_chain_sequence_id = fields.Many2one('ir.sequence', string='ZATCA account.move chain sequence',
                                                readonly=True, copy=False)

    l10n_sa_serial_number = fields.Char("Serial Number", copy=False,
                                        help="The serial number of the Taxpayer solution unit. Provided by ZATCA")

    l10n_sa_latest_submission_hash = fields.Char("Latest Submission Hash", copy=False,
                                                 help="Hash of the latest submitted invoice to be used as the Previous Invoice Hash (KSA-13)")

    # ====== Utility Functions =======

    def _l10n_sa_ready_to_submit_einvoices(self):
        """
            Helper function to know if the required CSIDs have been obtained, and the compliance checks have been
            completed
        """
        self.ensure_one()
        return self.sudo().l10n_sa_production_csid_json

    # ====== CSR Generation =======

    def _l10n_sa_csr_required_fields(self):
        """ Return the list of fields required to generate a valid CSR as per ZATCA requirements """
        return ['l10n_sa_private_key', 'vat', 'name', 'city', 'country_id', 'state_id']

    def _l10n_sa_get_csr_str(self):
        """
            Return s string representation of a ZATCA compliant CSR that will be sent to the Compliance API in order to get back
            a signed X509 certificate
        """
        self.ensure_one()

        company_id = self.company_id
        version_info = service.common.exp_version()
        builder = x509.CertificateSigningRequestBuilder()
        subject_names = (
            # Country Name
            (NameOID.COUNTRY_NAME, company_id.country_id.code),
            # Organization Unit Name
            (NameOID.ORGANIZATIONAL_UNIT_NAME, (company_id.vat or '')[:10]),
            # Organization Name
            (NameOID.ORGANIZATION_NAME, company_id.name),
            # Subject Common Name
            (NameOID.COMMON_NAME, company_id.name),
            # Organization Identifier
            (ObjectIdentifier('2.5.4.97'), company_id.vat),
            # State/Province Name
            (NameOID.STATE_OR_PROVINCE_NAME, company_id.state_id.name),
            # Locality Name
            (NameOID.LOCALITY_NAME, company_id.city),
        )
        # The CertificateSigningRequestBuilder instances are immutable, which is why everytime we modify one,
        # we have to assign it back to itself to keep track of the changes
        builder = builder.subject_name(x509.Name([
            x509.NameAttribute(n[0], u'%s' % n[1]) for n in subject_names
        ]))

        x509_alt_names_extension = x509.SubjectAlternativeName([
            x509.DirectoryName(x509.Name([
                # EGS Serial Number. Manufacturer or Solution Provider Name, Model or Version and Serial Number.
                # To be written in the following format: "1-... |2-... |3-..."
                x509.NameAttribute(ObjectIdentifier('2.5.4.4'), '1-Odoo|2-%s|3-%s' % (
                    version_info['server_version_info'][0], self.l10n_sa_serial_number)),
                # Organisation Identifier (UID)
                x509.NameAttribute(NameOID.USER_ID, company_id.vat),
                # Invoice Type. 4-digit numerical input using 0 & 1
                x509.NameAttribute(NameOID.TITLE, company_id._l10n_sa_get_csr_invoice_type()),
                # Location
                x509.NameAttribute(ObjectIdentifier('2.5.4.26'), company_id.street),
                # Industry
                x509.NameAttribute(ObjectIdentifier('2.5.4.15'), company_id.partner_id.industry_id.name or 'Other'),
            ]))
        ])

        x509_extensions = (
            # Add Certificate template name extension
            (x509.UnrecognizedExtension(ObjectIdentifier('1.3.6.1.4.1.311.20.2'),
                                        CERT_TEMPLATE_NAME[company_id.l10n_sa_api_mode]), False),
            # Add alternative names extension
            (x509_alt_names_extension, False),
        )

        for ext in x509_extensions:
            builder = builder.add_extension(ext[0], critical=ext[1])

        private_key = load_pem_private_key(company_id.l10n_sa_private_key, password=None, backend=default_backend())
        request = builder.sign(private_key, hashes.SHA256(), default_backend())

        return b64encode(request.public_bytes(Encoding.PEM)).decode()

    def _l10n_sa_generate_csr(self):
        """
            Generate a CSR for the Journal to be used for the Onboarding process and Invoice submissions
        """
        self.ensure_one()
        if any(not self.company_id[f] for f in self._l10n_sa_csr_required_fields()):
            raise UserError(_("Please, make sure all the following fields have been correctly set on the Company: \n")
                            + "\n".join(
                " - %s" % self.company_id._fields[f].string for f in self._l10n_sa_csr_required_fields() if
                not self.company_id[f]))
        self._l10n_sa_reset_certificates()
        self.l10n_sa_csr = self._l10n_sa_get_csr_str()

    # ====== Certificate Methods =======

    @api.depends('l10n_sa_production_csid_json')
    def _l10n_sa_compute_production_csid_validity(self):
        """
            Compute the expiration date of the Production certificate
        """
        for journal in self:
            journal.l10n_sa_production_csid_validity = False
            if journal.sudo().l10n_sa_production_csid_json:
                journal.l10n_sa_production_csid_validity = self._l10n_sa_get_pcsid_validity(
                    json.loads(journal.sudo().l10n_sa_production_csid_json)
                )

    def _l10n_sa_reset_certificates(self):
        """
            Reset all certificate values, including CSR and compliance checks
        """
        for journal in self.sudo():
            journal.l10n_sa_csr = False
            journal.l10n_sa_production_csid_json = False
            journal.l10n_sa_compliance_csid_json = False
            journal.l10n_sa_compliance_checks_passed = False

    def _l10n_sa_api_onboard_journal(self, otp):
        """
            Perform the onboarding for the journal. The onboarding consists of three steps:
                1.  Get the Compliance CSID
                2.  Perform the Compliance Checks
                3.  Get the Production CSID
        """
        self.ensure_one()
        try:
            # If the company does not have a private key, we generate it.
            # The private key is used to generate the CSR but also to sign the invoices
            if not self.company_id.l10n_sa_private_key:
                self.company_id.l10n_sa_private_key = self.company_id._l10n_sa_generate_private_key()
            self._l10n_sa_generate_csr()
            # STEP 1: The first step of the process is to get the CCSID
            self._l10n_sa_get_compliance_CSID(otp)
            # STEP 2: Once we have the CCSID, we preform the compliance checks
            self._l10n_sa_run_compliance_checks()
            # STEP 3: Once the compliance checks are completed, we request the PCSID
            self._l10n_sa_get_production_CSID()
            # Once all three steps are completed, we set the errors field to False
            self.l10n_sa_csr_errors = False
        except (RequestException, HTTPError, UserError) as e:
            # In case of an exception returned from ZATCA (not timeout), we will need to regenerate the CSR
            # As the same CSR cannot be used twice for the same CCSID request
            self._l10n_sa_reset_certificates()
            self.l10n_sa_csr_errors = e.args[0] or _("Journal could not be onboarded")

    def _l10n_sa_get_compliance_CSID(self, otp):
        """
            Request a Compliance Cryptographic Stamp Identifier (CCSID) from ZATCA
        """
        CCSID_data = self._l10n_sa_api_get_compliance_CSID(otp)
        if CCSID_data.get('error'):
            raise UserError(_("Could not obtain Compliance CSID: %s") % CCSID_data['error'])
        self.sudo().write({
            'l10n_sa_compliance_csid_json': json.dumps(CCSID_data),
            'l10n_sa_production_csid_json': False,
            'l10n_sa_compliance_checks_passed': False,
        })

    def _l10n_sa_get_production_CSID(self, OTP=None):
        """
            Request a Production Cryptographic Stamp Identifier (PCSID) from ZATCA
        """

        self_sudo = self.sudo()

        if not self_sudo.l10n_sa_compliance_csid_json:
            raise UserError(_("Cannot request a Production CSID before requesting a CCSID first"))
        elif not self_sudo.l10n_sa_compliance_checks_passed:
            raise UserError(_("Cannot request a Production CSID before completing the Compliance Checks"))

        renew = False
        zatca_format = self.env.ref('l10n_sa_edi.edi_sa_zatca')

        if self_sudo.l10n_sa_production_csid_json:
            time_now = zatca_format._l10n_sa_get_zatca_datetime(datetime.now())
            if zatca_format._l10n_sa_get_zatca_datetime(self_sudo.l10n_sa_production_csid_validity) < time_now:
                renew = True
            else:
                raise UserError(_("The Production CSID is still valid. You can only renew it once it has expired."))

        CCSID_data = json.loads(self_sudo.l10n_sa_compliance_csid_json)
        PCSID_data = self_sudo._l10n_sa_request_production_csid(CCSID_data, renew, OTP)
        if PCSID_data.get('error'):
            raise UserError(_("Could not obtain Production CSID: %s") % PCSID_data['error'])
        self_sudo.l10n_sa_production_csid_json = json.dumps(PCSID_data)

    # ====== Compliance Checks =======

    def _l10n_sa_get_compliance_files(self):
        """
            Return the list of files to be used for the compliance checks.
        """
        file_names, compliance_files = [
            'standard/invoice.xml', 'standard/credit.xml', 'standard/debit.xml',
            'simplified/invoice.xml', 'simplified/credit.xml', 'simplified/debit.xml',
        ], {}
        for file in file_names:
            fpath = get_module_resource('l10n_sa_edi', 'tests/compliance', file)
            with open(fpath, 'rb') as ip:
                compliance_files[file] = ip.read().decode()
        return compliance_files

    def _l10n_sa_run_compliance_checks(self):
        """
            Run Compliance Checks once the CCSID has been obtained.

            The goal of the Compliance Checks is to make sure our system is able to produce, sign and send Invoices
            correctly. For this we use dummy invoice UBL files available under the tests/compliance folder:

            Standard Invoice, Standard Credit Note, Standard Debit Note, Simplified Invoice, Simplified Credit Note,
            Simplified Debit Note.

            We read each one of these files separately, sign them, then process them through the Compliance Checks API.
        """

        self.ensure_one()
        self_sudo = self.sudo()
        if self.country_code != 'SA':
            raise UserError(_("Compliance checks can only be run for companies operating from KSA"))
        if not self_sudo.l10n_sa_compliance_csid_json:
            raise UserError(_("You need to request the CCSID first before you can proceed"))
        CCSID_data = json.loads(self_sudo.l10n_sa_compliance_csid_json)
        compliance_files = self._l10n_sa_get_compliance_files()
        for fname, fval in compliance_files.items():
            invoice_hash_hex = self.env['account.edi.xml.ubl_21.zatca']._l10n_sa_generate_invoice_xml_hash(
                fval).decode()
            digital_signature = self.env.ref('l10n_sa_edi.edi_sa_zatca')._l10n_sa_get_digital_signature(self.company_id, invoice_hash_hex).decode()
            prepared_xml = self._l10n_sa_prepare_compliance_xml(fname, fval, CCSID_data['binarySecurityToken'],
                                                                digital_signature)
            result = self._l10n_sa_api_compliance_checks(prepared_xml.decode(), CCSID_data)
            if result.get('error'):
                raise UserError(Markup("<p class='mb-0'>%s <b>%s</b></p>") % (_("Could not complete Compliance Checks for the following file:"), fname))
            if result['validationResults']['status'] == 'WARNING':
                warnings = "".join(Markup("<li><b>%s</b>: %s </li>") % (e['code'], e['message']) for e in result['validationResults']['warningMessages'])
                self.l10n_sa_csr_errors = Markup("<br/><br/><ul class='pl-3'><b>%s</b>%s</ul>") % (_("Warnings:"), warnings)
            elif result['validationResults']['status'] != 'PASS':
                errors = "".join(Markup("<li><b>%s</b>: %s </li>") % (e['code'], e['message']) for e in result['validationResults']['errorMessages'])
                raise UserError(Markup("<p class='mb-0'>%s <b>%s</b> %s</p>")
                                % (_("Could not complete Compliance Checks for the following file:"), fname, Markup("<br/><br/><ul class='pl-3'><b>%s</b>%s</ul>") % (_("Errors:"), errors)))
        self.l10n_sa_compliance_checks_passed = True

    def _l10n_sa_prepare_compliance_xml(self, xml_name, xml_raw, PCSID, signature):
        """
            Prepare XML content to be used for Compliance checks
        """
        xml_content = self._l10n_sa_prepare_invoice_xml(xml_raw)
        signed_xml = self.env.ref('l10n_sa_edi.edi_sa_zatca')._l10n_sa_sign_xml(xml_content, PCSID, signature)
        if xml_name.startswith('simplified'):
            qr_code_str = self.env['account.move']._l10n_sa_get_qr_code(self, signed_xml, b64decode(PCSID).decode(),
                                                                        signature, True)
            root = etree.fromstring(signed_xml)
            qr_node = root.xpath('//*[local-name()="ID"][text()="QR"]/following-sibling::*/*')[0]
            qr_node.text = b64encode(qr_code_str).decode()
            return etree.tostring(root, with_tail=False)
        return signed_xml

    def _l10n_sa_prepare_invoice_xml(self, xml_content):
        """
            Prepare the XML content of the test invoices before running the compliance checks
        """
        ubl_extensions = etree.fromstring(self.env['ir.qweb']._render('l10n_sa_edi.export_sa_zatca_ubl_extensions'))
        root = etree.fromstring(xml_content.encode())
        root.insert(0, ubl_extensions)
        ns_map = self.env['account.edi.xml.ubl_21.zatca']._l10n_sa_get_namespaces()

        def _get_node(xpath_str):
            return root.xpath(xpath_str, namespaces=ns_map)[0]

        # Update the Company VAT number in the test invoice
        vat_el = _get_node('//cbc:CompanyID')
        vat_el.text = self.company_id.vat

        # Update the Company Name in the test invoice
        name_nodes = ['cac:PartyName/cbc:Name', 'cac:PartyLegalEntity/cbc:RegistrationName', 'cac:Contact/cbc:Name']
        for node in name_nodes:
            comp_name_el = _get_node('//cac:AccountingSupplierParty/cac:Party/' + node)
            comp_name_el.text = self.company_id.display_name

        return etree.tostring(root)

    # ====== Index Chain & Previous Invoice Calculation =======

    def _l10n_sa_edi_get_next_chain_index(self):
        self.ensure_one()
        if not self.l10n_sa_chain_sequence_id:
            self.l10n_sa_chain_sequence_id = self.env['ir.sequence'].create({
                'name': f'ZATCA account move sequence for Journal {self.name} (id: {self.id})',
                'code': f'l10n_sa_edi.account.move.{self.id}',
                'implementation': 'no_gap',
                'company_id': self.company_id.id,
            })
        return self.l10n_sa_chain_sequence_id.next_by_id()

    def _l10n_sa_get_last_posted_invoice(self):
        """
        Returns the last invoice posted to this journal's chain.
        That invoice may have been received by the govt or not (eg. in case of a timeout).
        Only upon confirmed reception/refusal of that invoice can another one be posted.
        """
        self.ensure_one()
        return self.env['account.move'].search(
            [
                ('journal_id', '=', self.id),
                ('l10n_sa_chain_index', '!=', 0)
            ],
            limit=1, order='l10n_sa_chain_index desc'
        )

    # ====== API Calls to ZATCA =======

    def _l10n_sa_api_get_compliance_CSID(self, otp):
        """
            API call to the Compliance CSID API to generate a CCSID certificate, password and compliance request_id
            Requires a CSR token and a One Time Password (OTP)
        """
        self.ensure_one()
        if not otp:
            raise UserError(_("Please, set a valid OTP to be used for Onboarding"))
        if not self.l10n_sa_csr:
            raise UserError(_("Please, generate a CSR before requesting a CCSID"))
        request_data = {
            'body': json.dumps({'csr': self.l10n_sa_csr.decode()}),
            'header': {'OTP': otp}
        }
        return self._l10n_sa_call_api(request_data, ZATCA_API_URLS['apis']['ccsid'], 'POST')

    def _l10n_sa_api_get_production_CSID(self, CCSID_data):
        """
            API call to the Production CSID API to generate a PCSID certificate, password and production request_id
            Requires a requestID from the Compliance CSID API
        """
        request_data = {
            'body': json.dumps({'compliance_request_id': str(CCSID_data['requestID'])}),
            'header': {'Authorization': self._l10n_sa_authorization_header(CCSID_data)}
        }
        return self._l10n_sa_call_api(request_data, ZATCA_API_URLS['apis']['pcsid'], 'POST')

    def _l10n_sa_api_renew_production_CSID(self, PCSID_data, OTP):
        """
            API call to the Production CSID API to renew a PCSID certificate, password and production request_id
            Requires an expired Production CSIDPCSID_data
        """
        self.ensure_one()
        auth_data = PCSID_data
        # For renewal, the sandbox API expects a specific Username/Password, which are set in the SANDBOX_AUTH dict
        if self.company_id.l10n_sa_api_mode == 'sandbox':
            auth_data = SANDBOX_AUTH
        request_data = {
            'body': json.dumps({'csr': self.l10n_sa_csr.decode()}),
            'header': {
                'OTP': OTP,
                'Authorization': self._l10n_sa_authorization_header(auth_data)
            }
        }
        return self._l10n_sa_call_api(request_data, ZATCA_API_URLS['apis']['pcsid'], 'PATCH')

    def _l10n_sa_api_compliance_checks(self, xml_content, CCSID_data):
        """
            API call to the COMPLIANCE endpoint to generate a security token used for subsequent API calls
            Requires a CSR token and a One Time Password (OTP)
        """
        invoice_tree = etree.fromstring(xml_content)

        # Get the Invoice Hash from the XML document
        invoice_hash_node = invoice_tree.xpath('//*[@Id="invoiceSignedData"]/*[local-name()="DigestValue"]')[0]
        invoice_hash = invoice_hash_node.text

        # Get the Invoice UUID from the XML document
        invoice_uuid_node = invoice_tree.xpath('//*[local-name()="UUID"]')[0]
        invoice_uuid = invoice_uuid_node.text

        request_data = {
            'body': json.dumps({
                "invoiceHash": invoice_hash,
                "uuid": invoice_uuid,
                "invoice": b64encode(xml_content.encode()).decode()
            }),
            'header': {
                'Authorization': self._l10n_sa_authorization_header(CCSID_data),
                'Clearance-Status': '1'
            }
        }
        return self._l10n_sa_call_api(request_data, ZATCA_API_URLS['apis']['compliance'], 'POST')

    def _l10n_sa_get_api_clearance_url(self, invoice):
        """
            Return the API to be used for clearance. To be overridden to account for other cases, such as reporting.
        """
        return ZATCA_API_URLS['apis']['reporting' if invoice._l10n_sa_is_simplified() else 'clearance']

    def _l10n_sa_api_clearance(self, invoice, xml_content, PCSID_data):
        """
            API call to the CLEARANCE/REPORTING endpoint to sign an invoice
                - If SIMPLIFIED invoice: Reporting
                - If STANDARD invoice: Clearance
        """
        invoice_tree = etree.fromstring(xml_content)
        invoice_hash_node = invoice_tree.xpath('//*[@Id="invoiceSignedData"]/*[local-name()="DigestValue"]')[0]
        invoice_hash = invoice_hash_node.text
        request_data = {
            'body': json.dumps({
                "invoiceHash": invoice_hash,
                "uuid": invoice.l10n_sa_uuid,
                "invoice": b64encode(xml_content.encode()).decode()
            }),
            'header': {
                'Authorization': self._l10n_sa_authorization_header(PCSID_data),
                'Clearance-Status': '1'
            }
        }
        url_string = self._l10n_sa_get_api_clearance_url(invoice)
        return self._l10n_sa_call_api(request_data, url_string, 'POST')

    # ====== Certificate Methods =======

    def _l10n_sa_get_pcsid_validity(self, PCSID_data):
        """
            Return PCSID expiry date
        """
        b64_decoded_pcsid = b64decode(PCSID_data['binarySecurityToken'])
        x509_certificate = load_der_x509_certificate(b64decode(b64_decoded_pcsid.decode()), default_backend())
        return x509_certificate.not_valid_after

    def _l10n_sa_request_production_csid(self, csid_data, renew=False, otp=None):
        """
            Generate company Production CSID data
        """
        self.ensure_one()
        return (
            self._l10n_sa_api_renew_production_CSID(csid_data, otp)
            if renew
            else self._l10n_sa_api_get_production_CSID(csid_data)
        )

    def _l10n_sa_api_get_pcsid(self):
        """
            Get CSIDs required to perform ZATCA api calls, and regenerate them if they need to be regenerated.
        """
        self.ensure_one()
        if not self.sudo().l10n_sa_production_csid_json:
            raise UserError(_("Please, make a request to obtain the Compliance CSID and Production CSID before sending "
                            "documents to ZATCA"))
        pcsid_validity = self.env.ref('l10n_sa_edi.edi_sa_zatca')._l10n_sa_get_zatca_datetime(self.l10n_sa_production_csid_validity)
        time_now = self.env.ref('l10n_sa_edi.edi_sa_zatca')._l10n_sa_get_zatca_datetime(datetime.now())
        if pcsid_validity < time_now and self.company_id.l10n_sa_api_mode != 'sandbox':
            raise UserError(_("Production certificate has expired, please renew the PCSID before proceeding"))
        return json.loads(self.sudo().l10n_sa_production_csid_json)

    # ====== API Helper Methods =======

    def _l10n_sa_call_api(self, request_data, request_url, method):
        """
            Helper function to make api calls to the ZATCA API Endpoint
        """
        api_url = ZATCA_API_URLS[self.company_id.l10n_sa_api_mode]
        request_url = urljoin(api_url, request_url)
        status_code = False
        try:
            request_response = requests.request(method, request_url, data=request_data.get('body'),
                                                headers={
                                                    **self._l10n_sa_api_headers(),
                                                    **request_data.get('header')
                                                }, timeout=(30, 30))
            request_response.raise_for_status()
        except (ValueError, HTTPError) as ex:
            # The 400 case means that it is rejected by ZATCA, but we need to update the hash as done for accepted.
            # In the 401+ cases, it is like the server is overloaded e.g. and we still need to resend later.  We do not
            # erase the index chain (excepted) because for ZATCA, one ICV (index chain) needs to correspond to one invoice.
            status_code = ex.response.status_code
            if status_code != 400:
                return {
                    'error': (Markup("<b>[%s]</b>") % status_code) + _("Server returned an unexpected error: %(error)s",
                               error=(request_response.text or str(ex))),
                    'blocking_level': 'warning',
                    'status_code': status_code,
                    'excepted': True,
                }
        except RequestException as ex:
            # Usually only happens if a Timeout occurs. In this case we're not sure if the invoice was accepted or
            # rejected, or if it even made it to ZATCA
            return {'error': str(ex), 'blocking_level': 'warning', 'excepted': True}

        if request_response.status_code == '303':
            return {'error': _('Clearance and reporting seem to have been mixed up. '),
                    'blocking_level': 'warning', 'excepted': True}

        try:
            response_data = request_response.json()
        except json.decoder.JSONDecodeError:
            return {
                'error': _("JSON response from ZATCA could not be decoded"),
                'blocking_level': 'error'
            }
        response_data['status_code'] = request_response.status_code

        val_res = response_data.get('validationResults', {})
        if not request_response.ok and (val_res.get('errorMessages') or val_res.get('warningMessages')):
            error = "" if not status_code else Markup("<b>[%s]</b>") % (status_code)
            if isinstance(response_data, dict) and val_res.get('errorMessages'):
                error += _("Invoice submission to ZATCA returned errors")
                return {
                    'error': error,
                    'json_errors': response_data,
                    'blocking_level': 'error',
                }
            error += request_response.reason
            return {
                'error': error,
                'blocking_level': 'error',
            }
        return response_data

    def _l10n_sa_api_headers(self):
        """
            Return the base headers to be included in ZATCA API calls
        """
        return {
            'Content-Type': 'application/json',
            'Accept-Language': 'en',
            'Accept-Version': 'V2'
        }

    def _l10n_sa_authorization_header(self, CSID_data):
        """
            Compute the Authorization header by combining the CSID and the Secret key, then encode to Base64
        """
        auth_data = CSID_data
        auth_str = "%s:%s" % (auth_data['binarySecurityToken'], auth_data['secret'])
        return 'Basic ' + b64encode(auth_str.encode()).decode()

    def _l10n_sa_load_edi_demo_data(self):
        self.ensure_one()
        self.company_id.l10n_sa_private_key = self.company_id._l10n_sa_generate_private_key()
        self.write({
            'l10n_sa_serial_number': 'SIDI3-CBMPR-L2D8X-KM0KN-X4ISJ',
            'l10n_sa_compliance_checks_passed': True,
            'l10n_sa_csr': b'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ2NqQ0NBaGNDQVFBd2djRXhDekFKQmdOVkJBWVRBbE5CTVJNd0VRWURWUVFMREFvek1UQXhOelV6T1RjMApNUk13RVFZRFZRUUtEQXBUUVNCRGIyMXdZVzU1TVJNd0VRWURWUVFEREFwVFFTQkRiMjF3WVc1NU1SZ3dGZ1lEClZRUmhEQTh6TVRBeE56VXpPVGMwTURBd01ETXhEekFOQmdOVkJBZ01CbEpwZVdGa2FERklNRVlHQTFVRUJ3dy8KdzVqQ3A4T1o0b0NldzVuaWdLYkRtTUt2dzVuRm9NT1o0b0NndzVqQ3FTRERtTUtudzVuaWdKN0RtZUtBcHNPWgo0b0NndzVuTGhzT1l3ckhEbU1LcE1GWXdFQVlIS29aSXpqMENBUVlGSzRFRUFBb0RRZ0FFN2ZpZWZWQ21HcTlzCmV0OVl4aWdQNzZWUmJxZlh0VWNtTk1VN3FkTlBiSm5NNGh5R1QwanpPcXUrSWNXWW5IelFJYmxJVmsydENPQnQKYjExanY4MGVwcUNCOVRDQjhnWUpLb1pJaHZjTkFRa09NWUhrTUlIaE1DUUdDU3NHQVFRQmdqY1VBZ1FYRXhWUQpVa1ZhUVZSRFFTMURiMlJsTFZOcFoyNXBibWN3Z2JnR0ExVWRFUVNCc0RDQnJhU0JxakNCcHpFME1ESUdBMVVFCkJBd3JNUzFQWkc5dmZESXRNVFY4TXkxVFNVUkpNeTFEUWsxUVVpMU1Na1E0V0MxTFRUQkxUaTFZTkVsVFNqRWYKTUIwR0NnbVNKb21UOGl4a0FRRU1Eek14TURFM05UTTVOelF3TURBd016RU5NQXNHQTFVRURBd0VNVEV3TURFdgpNQzBHQTFVRUdnd21RV3dnUVcxcGNpQk5iMmhoYlcxbFpDQkNhVzRnUVdKa2RXd2dRWHBwZWlCVGRISmxaWFF4CkRqQU1CZ05WQkE4TUJVOTBhR1Z5TUFvR0NDcUdTTTQ5QkFNQ0Ewa0FNRVlDSVFEb3VCeXhZRDRuQ2pUQ2V6TkYKczV6SmlVWW1QZVBRNnFWNDdZemRHeWRla1FJaEFPRjNVTWF4UFZuc29zOTRFMlNkT2JJcTVYYVAvKzlFYWs5TgozMUtWRUkvTQotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K',
            'l10n_sa_compliance_csid_json': """{"requestID": 1234567890123, "dispositionMessage": "ISSUED", "binarySecurityToken": "TUlJQ2N6Q0NBaG1nQXdJQkFnSUdBWStWTmxza01Bb0dDQ3FHU000OUJBTUNNQlV4RXpBUkJnTlZCQU1NQ21WSmJuWnZhV05wYm1jd0hoY05NalF3TlRJd01EZzFOVEV6V2hjTk1qa3dOVEU1TWpFd01EQXdXakNCbnpFTE1Ba0dBMVVFQmhNQ1UwRXhFekFSQmdOVkJBc01Dak01T1RrNU9UazVPVGt4RXpBUkJnTlZCQW9NQ2xOQklFTnZiWEJoYm5reEV6QVJCZ05WQkFNTUNsTkJJRU52YlhCaGJua3hHREFXQmdOVkJHRU1Eek01T1RrNU9UazVPVGt3TURBd016RVBNQTBHQTFVRUNBd0dVbWw1WVdSb01TWXdKQVlEVlFRSERCM1lwOW1FMllYWXI5bUsyWWJZcVNEWXA5bUUyWVhaaHRtSTJMSFlxVEJXTUJBR0J5cUdTTTQ5QWdFR0JTdUJCQUFLQTBJQUJOVlB3N0hGNjhUVWtQTkJQb29uT0Y2NnRPMm5IcmxUNlRMcmk3MEpLY1MvYmVMWitoRVE0MmdXdUtYckp5RmxnWm9kUVJzTFQyMEtQZnE0Q3N2YlFJMmpnY3d3Z2Nrd0RBWURWUjBUQVFIL0JBSXdBRENCdUFZRFZSMFJCSUd3TUlHdHBJR3FNSUduTVRRd01nWURWUVFFRENzeExVOWtiMjk4TWkweE5Yd3pMVk5KUkVrekxVTkNUVkJTTFV3eVJEaFlMVXROTUV0T0xWZzBTVk5LTVI4d0hRWUtDWkltaVpQeUxHUUJBUXdQTXprNU9UazVPVGs1T1RBd01EQXpNUTB3Q3dZRFZRUU1EQVF4TVRBd01TOHdMUVlEVlFRYURDWkJiQ0JCYldseUlFMXZhR0Z0YldWa0lFSnBiaUJCWW1SMWJDQkJlbWw2SUZOMGNtVmxkREVPTUF3R0ExVUVEd3dGVDNSb1pYSXdDZ1lJS29aSXpqMEVBd0lEU0FBd1JRSWdTeVhlZExqOUtMVTRUMWFBbVQvL09GZDBGWWxLQnIraFFIeGNDM0c2ajc4Q0lRRGdlNjNsQkVqTU1ETktqTm1pTklaQlBWSnlHRzl5bVJaSHdvUzV5TEQyZXc9PQ==", "secret": "uMpSz85cV0h/e/uqpJ+FaZkdYZ76uoaRYOevGufcup0=", "errors": null}""",
            'l10n_sa_production_csid_json': """{"requestID": 30368, "tokenType": "http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-x509-token-profile-1.0#X509v3", "dispositionMessage": "ISSUED", "binarySecurityToken": "TUlJRDNqQ0NBNFNnQXdJQkFnSVRFUUFBT0FQRjkwQWpzL3hjWHdBQkFBQTRBekFLQmdncWhrak9QUVFEQWpCaU1SVXdFd1lLQ1pJbWlaUHlMR1FCR1JZRmJHOWpZV3d4RXpBUkJnb0praWFKay9Jc1pBRVpGZ05uYjNZeEZ6QVZCZ29Ka2lhSmsvSXNaQUVaRmdkbGVIUm5ZWHAwTVJzd0dRWURWUVFERXhKUVVscEZTVTVXVDBsRFJWTkRRVFF0UTBFd0hoY05NalF3TVRFeE1Ea3hPVE13V2hjTk1qa3dNVEE1TURreE9UTXdXakIxTVFzd0NRWURWUVFHRXdKVFFURW1NQ1FHQTFVRUNoTWRUV0Y0YVcxMWJTQlRjR1ZsWkNCVVpXTm9JRk4xY0hCc2VTQk1WRVF4RmpBVUJnTlZCQXNURFZKcGVXRmthQ0JDY21GdVkyZ3hKakFrQmdOVkJBTVRIVlJUVkMwNE9EWTBNekV4TkRVdE16azVPVGs1T1RrNU9UQXdNREF6TUZZd0VBWUhLb1pJemowQ0FRWUZLNEVFQUFvRFFnQUVvV0NLYTBTYTlGSUVyVE92MHVBa0MxVklLWHhVOW5QcHgydmxmNHloTWVqeThjMDJYSmJsRHE3dFB5ZG84bXEwYWhPTW1Obzhnd25pN1h0MUtUOVVlS09DQWdjd2dnSURNSUd0QmdOVkhSRUVnYVV3Z2FLa2daOHdnWnd4T3pBNUJnTlZCQVFNTWpFdFZGTlVmREl0VkZOVWZETXRaV1F5TW1ZeFpEZ3RaVFpoTWkweE1URTRMVGxpTlRndFpEbGhPR1l4TVdVME5EVm1NUjh3SFFZS0NaSW1pWlB5TEdRQkFRd1BNems1T1RrNU9UazVPVEF3TURBek1RMHdDd1lEVlFRTURBUXhNVEF3TVJFd0R3WURWUVFhREFoU1VsSkVNamt5T1RFYU1CZ0dBMVVFRHd3UlUzVndjR3g1SUdGamRHbDJhWFJwWlhNd0hRWURWUjBPQkJZRUZFWCtZdm1tdG5Zb0RmOUJHYktvN29jVEtZSzFNQjhHQTFVZEl3UVlNQmFBRkp2S3FxTHRtcXdza0lGelZ2cFAyUHhUKzlObk1Ic0dDQ3NHQVFVRkJ3RUJCRzh3YlRCckJnZ3JCZ0VGQlFjd0FvWmZhSFIwY0RvdkwyRnBZVFF1ZW1GMFkyRXVaMjkyTG5OaEwwTmxjblJGYm5KdmJHd3ZVRkphUlVsdWRtOXBZMlZUUTBFMExtVjRkR2RoZW5RdVoyOTJMbXh2WTJGc1gxQlNXa1ZKVGxaUFNVTkZVME5CTkMxRFFTZ3hLUzVqY25Rd0RnWURWUjBQQVFIL0JBUURBZ2VBTUR3R0NTc0dBUVFCZ2pjVkJ3UXZNQzBHSlNzR0FRUUJnamNWQ0lHR3FCMkUwUHNTaHUyZEpJZk8reG5Ud0ZWbWgvcWxaWVhaaEQ0Q0FXUUNBUkl3SFFZRFZSMGxCQll3RkFZSUt3WUJCUVVIQXdNR0NDc0dBUVVGQndNQ01DY0dDU3NHQVFRQmdqY1ZDZ1FhTUJnd0NnWUlLd1lCQlFVSEF3TXdDZ1lJS3dZQkJRVUhBd0l3Q2dZSUtvWkl6ajBFQXdJRFNBQXdSUUloQUxFL2ljaG1uV1hDVUtVYmNhM3ljaThvcXdhTHZGZEhWalFydmVJOXVxQWJBaUE5aEM0TThqZ01CQURQU3ptZDJ1aVBKQTZnS1IzTEUwM1U3NWVxYkMvclhBPT0=", "secret": "CkYsEXfV8c1gFHAtFWoZv73pGMvh/Qyo4LzKM2h/8Hg="}"""
        })

```

## File: models\account_move.py

```python
import uuid
import json
from markupsafe import Markup
from odoo import _, fields, models, api
from odoo.tools import float_repr
from datetime import datetime
from base64 import b64decode, b64encode
from lxml import etree
from cryptography.hazmat.primitives.serialization import Encoding, PublicFormat
from cryptography.hazmat.backends import default_backend
from cryptography.x509 import load_der_x509_certificate


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_sa_uuid = fields.Char(string='Document UUID (SA)', copy=False, help="Universally unique identifier of the Invoice")

    l10n_sa_invoice_signature = fields.Char("Unsigned XML Signature", copy=False)

    l10n_sa_chain_index = fields.Integer(
        string="ZATCA chain index", copy=False, readonly=True,
        help="Invoice index in chain, set if and only if an in-chain XML was submitted and did not error",
    )

    def _l10n_sa_is_simplified(self):
        """
            Returns True if the customer is an individual, i.e: The invoice is B2C
        :return:
        """
        self.ensure_one()
        return self.partner_id.company_type == 'person'

    @api.depends('amount_total_signed', 'amount_tax_signed', 'l10n_sa_confirmation_datetime', 'company_id',
                 'company_id.vat', 'journal_id', 'journal_id.l10n_sa_production_csid_json', 'edi_document_ids',
                 'l10n_sa_invoice_signature', 'l10n_sa_chain_index', 'state')
    def _compute_qr_code_str(self):
        """ Override to update QR code generation in accordance with ZATCA Phase 2"""
        phase_one_moves = self.env['account.move']
        for move in self:
            zatca_document = move.edi_document_ids.filtered(lambda d: d.edi_format_id.code == 'sa_zatca')
            if move.country_code == 'SA' and move.move_type in ('out_invoice', 'out_refund') and zatca_document and move.state != 'draft':
                qr_code_str = ''
                if move._l10n_sa_is_simplified():
                    x509_cert = json.loads(move.journal_id.sudo().l10n_sa_production_csid_json)['binarySecurityToken']
                    xml_content = self.env.ref('l10n_sa_edi.edi_sa_zatca')._l10n_sa_generate_zatca_template(move)
                    qr_code_str = move._l10n_sa_get_qr_code(move.journal_id, xml_content, b64decode(x509_cert),
                                                            move.l10n_sa_invoice_signature, True)
                    qr_code_str = b64encode(qr_code_str).decode()
                elif zatca_document.state == 'sent' and zatca_document.sudo().attachment_id.datas:
                    document_xml = zatca_document.attachment_id.with_context(bin_size=False).datas.decode()
                    root = etree.fromstring(b64decode(document_xml))
                    qr_node = root.xpath('//*[local-name()="ID"][text()="QR"]/following-sibling::*/*')[0]
                    qr_code_str = qr_node.text
                move.l10n_sa_qr_code_str = qr_code_str
            else:
                # In the case where the Invoice is not a ZATCA invoice, or is Phase 1, or is not confirmed,
                # we call super to trigger the initial QR code generation for Phase 1
                phase_one_moves |= move
        super(AccountMove, phase_one_moves)._compute_qr_code_str()


    def _l10n_sa_get_qr_code_encoding(self, tag, field, int_length=1):
        """
            Helper function to encode strings for the QR code generation according to ZATCA specs
        """
        company_name_tag_encoding = tag.to_bytes(length=1, byteorder='big')
        company_name_length_encoding = len(field).to_bytes(length=int_length, byteorder='big')
        return company_name_tag_encoding + company_name_length_encoding + field

    def _l10n_sa_check_refund_reason(self):
        """
            Make sure credit/debit notes have a valid reason and reversal reference
        """
        self.ensure_one()
        return self.reversed_entry_id or self.ref

    @api.model
    def _l10n_sa_get_qr_code(self, journal_id, unsigned_xml, x509_cert, signature, is_b2c=False):
        """
            Generate QR code string based on XML content of the Invoice UBL file, X509 Production Certificate
            and company info.

            :return b64 encoded QR code string
        """

        def xpath_ns(expr):
            return root.xpath(expr, namespaces=edi_format._l10n_sa_get_namespaces())[0].text.strip()

        qr_code_str = b''
        root = etree.fromstring(unsigned_xml)
        edi_format = self.env['account.edi.xml.ubl_21.zatca']

        # Indent XML content to avoid indentation mismatches
        etree.indent(root, space='    ')

        invoice_date = xpath_ns('//cbc:IssueDate')
        invoice_time = xpath_ns('//cbc:IssueTime')
        invoice_datetime = datetime.strptime(invoice_date + ' ' + invoice_time, '%Y-%m-%d %H:%M:%S')

        if invoice_datetime and journal_id.company_id.vat and x509_cert and signature:
            prehash_content = etree.tostring(root)
            invoice_hash = edi_format._l10n_sa_generate_invoice_xml_hash(prehash_content, 'digest')

            amount_total = float(xpath_ns('//cbc:TaxInclusiveAmount'))
            amount_tax = float(xpath_ns('//cac:TaxTotal/cbc:TaxAmount'))
            x509_certificate = load_der_x509_certificate(b64decode(x509_cert), default_backend())
            seller_name_enc = self._l10n_sa_get_qr_code_encoding(1, journal_id.company_id.display_name.encode())
            seller_vat_enc = self._l10n_sa_get_qr_code_encoding(2, journal_id.company_id.vat.encode())
            timestamp_enc = self._l10n_sa_get_qr_code_encoding(3,
                                                               invoice_datetime.strftime("%Y-%m-%dT%H:%M:%S").encode())
            amount_total_enc = self._l10n_sa_get_qr_code_encoding(4, float_repr(abs(amount_total), 2).encode())
            amount_tax_enc = self._l10n_sa_get_qr_code_encoding(5, float_repr(abs(amount_tax), 2).encode())
            invoice_hash_enc = self._l10n_sa_get_qr_code_encoding(6, invoice_hash)
            signature_enc = self._l10n_sa_get_qr_code_encoding(7, signature.encode())
            public_key_enc = self._l10n_sa_get_qr_code_encoding(8,
                                                                x509_certificate.public_key().public_bytes(Encoding.DER,
                                                                                                           PublicFormat.SubjectPublicKeyInfo))

            qr_code_str = (seller_name_enc + seller_vat_enc + timestamp_enc + amount_total_enc +
                           amount_tax_enc + invoice_hash_enc + signature_enc + public_key_enc)

            if is_b2c:
                qr_code_str += self._l10n_sa_get_qr_code_encoding(9, x509_certificate.signature)

        return qr_code_str

    @api.depends('state', 'edi_document_ids.state')
    def _compute_edi_show_cancel_button(self):
        """
            Override to hide the EDI Cancellation button at all times for ZATCA Invoices
        """
        super()._compute_edi_show_cancel_button()
        for move in self.filtered(lambda m: m.is_invoice() and m.country_code == 'SA'):
            move.edi_show_cancel_button = False

    @api.depends('state', 'edi_document_ids.state')
    def _compute_show_reset_to_draft_button(self):
        """
            Override to hide the Reset to Draft button for ZATCA Invoices that have been successfully submitted
        """
        super()._compute_show_reset_to_draft_button()
        for move in self:
            # An invoice should only have an index chain if it was successfully submitted without rejection,
            # or if the submission timed out. In both cases, a user should not be able to reset it to draft.
            if move.l10n_sa_chain_index:
                move.show_reset_to_draft_button = False

    def _l10n_sa_reset_confirmation_datetime(self):
        """ OVERRIDE: we want rejected phase 2 invoices to keep the original confirmation datetime"""
        for move in self.filtered(lambda m: m.country_code == 'SA'):
            zatca_doc = move.edi_document_ids.filtered(lambda d: d.edi_format_id.code == 'sa_zatca')
            if not zatca_doc or zatca_doc[0].blocking_level != 'error':  # Error is the rejection case
                move.l10n_sa_confirmation_datetime = False

    def _l10n_sa_generate_unsigned_data(self):
        """
            Generate UUID and digital signature to be used during both Signing and QR code generation.
            It is necessary to save the signature as it changes everytime it is generated and both the signing and the
            QR code expect to have the same, identical signature.
        """
        self.ensure_one()
        edi_format = self.env.ref('l10n_sa_edi.edi_sa_zatca')
        # Build the dict of values to be used for generating the Invoice XML content
        # Set Invoice field values required for generating the XML content, hash and signature
        self.l10n_sa_uuid = uuid.uuid4()
        # We generate the XML content
        xml_content = edi_format._l10n_sa_generate_zatca_template(self)
        # Once the required values are generated, we hash the invoice, then use it to generate a Signature
        invoice_hash_hex = self.env['account.edi.xml.ubl_21.zatca']._l10n_sa_generate_invoice_xml_hash(xml_content).decode()
        self.l10n_sa_invoice_signature = edi_format._l10n_sa_get_digital_signature(self.journal_id.company_id,
                                                                                   invoice_hash_hex).decode()
        return xml_content

    def _l10n_sa_log_results(self, xml_content, response_data=None, error=False):
        """
            Save submitted invoice XML hash in case of either Rejection or Acceptance.
        """
        self.ensure_one()
        self.journal_id.l10n_sa_latest_submission_hash = self.env['account.edi.xml.ubl_21.zatca']._l10n_sa_generate_invoice_xml_hash(
            xml_content)
        bootstrap_cls, title, content = ("success", _("Invoice Successfully Submitted to ZATCA"),
                                         "" if (not error or not response_data) else response_data)
        attachment = False
        if error:
            xml_filename = self.env['account.edi.xml.ubl_21.zatca']._export_invoice_filename(self)
            xml_filename = xml_filename[:-4] + '-rejected.xml'
            attachment = self.env['ir.attachment'].create({
                'raw': xml_content,
                'name': xml_filename,
                'description': 'Rejected ZATCA Document not to be deleted - ثيقة ZATCA المرفوضة لا يجوز حذفها',
                'res_id': self.id,
                'res_model': self._name,
                'type': 'binary',
                'mimetype': 'application/xml',
            })
            bootstrap_cls, title = ("danger", _("Invoice was rejected by ZATCA"))
            error_msg = response_data['error']
            content = Markup("""
                <p class='mb-0'>
                    %s
                </p>
                <hr>
                <p class='mb-0'>
                    %s
                </p>
            """) % (_('The invoice was rejected by ZATCA. Please, check the response below:'), error_msg)
        if response_data and response_data.get('validationResults', {}).get('warningMessages'):
            status_code = response_data.get('status_code')
            bootstrap_cls, title = ("warning", _("Invoice was Accepted by ZATCA (with Warnings)"))
            content = Markup("""
                <p class='mb-0'>
                    %s
                </p>
                <hr>
                <p class='mb-0'>
                    <b>%s</b>%s
                </p>
            """) % (_('The invoice was accepted by ZATCA, but returned warnings. Please, check the response below:'),
                    f"[{status_code}] " if status_code else "",
                    Markup("<br/>").join([Markup("<b>%s</b> : %s") % (m['code'], m['message']) for m in response_data['validationResults']['warningMessages']]))
        self.with_context(no_new_invoice=True).message_post(body=Markup("""
                <div role='alert' class='alert alert-%s'>
                    <h4 class='alert-heading'>%s</h4>%s
                </div>
            """) % (bootstrap_cls, title, content),
            attachment_ids=attachment and [attachment.id] or []
        )

    def _is_l10n_sa_eligibile_invoice(self):
        self.ensure_one()
        return self.is_invoice() and self.l10n_sa_confirmation_datetime and self.country_code == 'SA'

    def _get_report_base_filename(self):
        """
            Generate the name of the invoice PDF file according to ZATCA business rules:
            Seller Vat Number (BT-31), Date (BT-2), Time (KSA-25), Invoice Number (BT-1)
        """
        if self._is_l10n_sa_eligibile_invoice():
            return self.with_context(l10n_sa_file_format=False).env['account.edi.xml.ubl_21.zatca']._export_invoice_filename(self)
        return super()._get_report_base_filename()

    def _get_report_attachment_filename(self):
        if self._is_l10n_sa_eligibile_invoice():
            return self.with_context(l10n_sa_file_format='pdf').env['account.edi.xml.ubl_21.zatca']._export_invoice_filename(self)
        return super()._get_report_attachment_filename()

    def _get_report_mail_attachment_filename(self):
        if self._is_l10n_sa_eligibile_invoice():
            return self.with_context(l10n_sa_file_format=False).env['account.edi.xml.ubl_21.zatca']._export_invoice_filename(self)
        return super()._get_report_mail_attachment_filename()

    def _l10n_sa_is_in_chain(self):
        """
        If the invoice was successfully posted and confirmed by the government, then this would return True.
        If the invoice timed out, then its edi_document should still be in the 'to_send' state.
        """
        zatca_doc_ids = self.edi_document_ids.filtered(lambda d: d.edi_format_id.code == 'sa_zatca')
        return len(zatca_doc_ids) > 0 and not any(zatca_doc_ids.filtered(lambda d: d.state == 'to_send'))

    def _get_tax_lines_to_aggregate(self):
        """
        If the final invoice has downpayment lines, we skip the tax correction, as we need to recalculate tax amounts
        without taking into account those lines
        """
        if self.country_code == 'SA' and not self._is_downpayment() and self.line_ids._get_downpayment_lines():
            return self.env['account.move.line']
        return super()._get_tax_lines_to_aggregate()

    def _get_l10n_sa_totals(self):
        self.ensure_one()
        invoice_vals = self.env['account.edi.xml.ubl_21.zatca']._export_invoice_vals(self)
        return {
            'total_amount': invoice_vals['vals']['legal_monetary_total_vals']['tax_inclusive_amount'],
            'total_tax': invoice_vals['vals']['tax_total_vals'][-1]['tax_amount'],
        }


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    def _apply_retention_tax_filter(self, tax_values):
        return not tax_values['tax_id'].l10n_sa_is_retention

    def _is_global_discount_line(self):
        """
            Any line that has a negative amount and is not linked to a down-payment is considered as a
            global discount line. These can be created either manually, or through a promotions program.
        """
        self.ensure_one()
        return not self._get_downpayment_lines() and self.price_subtotal < 0

    @api.depends('price_subtotal', 'price_total')
    def _compute_tax_amount(self):
        super()._compute_tax_amount()
        taxes_vals_by_move = {}
        for record in self:
            move = record.move_id
            if move.country_code == 'SA':
                taxes_vals = taxes_vals_by_move.get(move.id)
                if not taxes_vals:
                    taxes_vals = move._prepare_invoice_aggregated_taxes(
                        filter_tax_values_to_apply=lambda l, t: not self.env['account.tax'].browse(t['id']).l10n_sa_is_retention
                    )
                    taxes_vals_by_move[move.id] = taxes_vals
                record.l10n_gcc_invoice_tax_amount = abs(taxes_vals.get('tax_details_per_record', {}).get(record, {}).get('tax_amount_currency', 0))

```

## File: models\account_tax.py

```python
from odoo import fields, models, api, _
from odoo.exceptions import UserError


EXEMPTION_REASON_CODES = [
    ('VATEX-SA-29', 'VATEX-SA-29 Financial services mentioned in Article 29 of the VAT Regulations.'),
    ('VATEX-SA-29-7', 'VATEX-SA-29-7 Life insurance services mentioned in Article 29 of the VAT.'),
    ('VATEX-SA-30', 'VATEX-SA-30 Real estate transactions mentioned in Article 30 of the VAT Regulations.'),
    ('VATEX-SA-32', 'VATEX-SA-32 Export of goods.'),
    ('VATEX-SA-33', 'VATEX-SA-33 Export of Services.'),
    ('VATEX-SA-34-1', 'VATEX-SA-34-1 The international transport of Goods.'),
    ('VATEX-SA-34-2', 'VATEX-SA-34-1 The international transport of Passengers.'),
    ('VATEX-SA-34-3', 'VATEX-SA-34-3 Services directly connected and incidental to a Supply of international passenger transport.'),
    ('VATEX-SA-34-4', 'VATEX-SA-34-4 Supply of a qualifying means of transport.'),
    ('VATEX-SA-34-5', 'VATEX-SA-34-5 Any services relating to Goods or passenger transportation, as defined in article twenty five of these Regulations.'),
    ('VATEX-SA-35', 'VATEX-SA-35 Medicines and medical equipment.'),
    ('VATEX-SA-36', 'VATEX-SA-36 Qualifying metals.'),
    ('VATEX-SA-EDU', 'VATEX-SA-EDU Private education to citizen.'),
    ('VATEX-SA-HEA', 'VATEX-SA-HEA Private healthcare to citizen.'),
    ('VATEX-SA-OOS', 'VATEX-SA-OOS Not subject to VAT.')
]


class AccountTax(models.Model):
    _inherit = 'account.tax'

    l10n_sa_is_retention = fields.Boolean("Is Retention", default=False,
                                          help="Determines whether or not a tax counts as a Withholding Tax")

    l10n_sa_exemption_reason_code = fields.Selection(string="Exemption Reason Code",
                                                     selection=EXEMPTION_REASON_CODES, help="Tax Exemption Reason Code (ZATCA)")

    @api.onchange('amount')
    def onchange_amount(self):
        super().onchange_amount()
        self.l10n_sa_is_retention = False

    @api.constrains("l10n_sa_is_retention", "amount", "type_tax_use")
    def _l10n_sa_constrain_is_retention(self):
        for tax in self:
            if tax.amount >= 0 and tax.l10n_sa_is_retention and tax.type_tax_use == 'sale':
                raise UserError(_("Cannot set a tax to Retention if the amount is greater than or equal 0"))


class AccountTaxTemplate(models.Model):
    _inherit = 'account.tax.template'

    l10n_sa_is_retention = fields.Boolean("Is Retention", default=False,
                                          help="Determines whether or not a tax counts as a Withholding Tax")

    l10n_sa_exemption_reason_code = fields.Selection(string="Exemption Reason Code",
                                                     selection=EXEMPTION_REASON_CODES, help="Tax Exemption Reason Code (ZATCA)")

    def _get_tax_vals(self, company, tax_template_to_tax):
        # OVERRIDE
        res = super()._get_tax_vals(company, tax_template_to_tax)
        res['l10n_sa_is_retention'] = self.l10n_sa_is_retention
        res['l10n_sa_exemption_reason_code'] = self.l10n_sa_exemption_reason_code
        return res

```

## File: models\ir_attachment.py

```python
from odoo import api, models, _
from odoo.exceptions import UserError


class IrAttachment(models.Model):
    _inherit = 'ir.attachment'

    @api.ondelete(at_uninstall=False)
    def _unlink_except_rejected_zatca_document(self):
        '''
        Prevents unlinking of rejected XML documents
        '''
        descr = 'Rejected ZATCA Document not to be deleted - ثيقة ZATCA المرفوضة لا يجوز حذفها'
        for attach in self.filtered(lambda a: a.description == descr and a.res_model == 'account.move'):
            move = self.env['account.move'].browse(attach.res_id)
            if move.country_code == "SA":
                raise UserError(_("You can't unlink an attachment being an EDI document refused by the government."))

```

## File: models\res_company.py

```python
import re
from odoo import models, fields, _
from odoo.exceptions import UserError
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import ec


class ResCompany(models.Model):
    _inherit = "res.company"

    def _l10n_sa_generate_private_key(self):
        """
            Compute a private key for each company that will be used to generate certificate signing requests (CSR)
            in order to receive X509 certificates from the ZATCA APIs and sign EDI documents

            -   public_exponent=65537 is a default value that should be used most of the time, as per the documentation
                of cryptography.
            -   key_size=2048 is considered a reasonable default key size, as per the documentation of cryptography.

            See https://cryptography.io/en/latest/hazmat/primitives/asymmetric/ec/
        """
        private_key = ec.generate_private_key(ec.SECP256K1, default_backend())
        return private_key.private_bytes(
            encoding=serialization.Encoding.PEM,
            format=serialization.PrivateFormat.TraditionalOpenSSL,
            encryption_algorithm=serialization.NoEncryption())

    l10n_sa_private_key = fields.Binary("ZATCA Private key", attachment=False, groups="base.group_system", copy=False,
                                        help="The private key used to generate the CSR and obtain certificates",)

    l10n_sa_api_mode = fields.Selection(
        [('sandbox', 'Sandbox'), ('preprod', 'Simulation (Pre-Production)'), ('prod', 'Production')],
        help="Specifies which API the system should use", required=True,
        default='sandbox', copy=False)

    l10n_sa_edi_building_number = fields.Char(compute='_compute_address',
                                              inverse='_l10n_sa_edi_inverse_building_number')
    l10n_sa_edi_plot_identification = fields.Char(compute='_compute_address',
                                                  inverse='_l10n_sa_edi_inverse_plot_identification')

    l10n_sa_additional_identification_scheme = fields.Selection(
        related='partner_id.l10n_sa_additional_identification_scheme', readonly=False)
    l10n_sa_additional_identification_number = fields.Char(
        related='partner_id.l10n_sa_additional_identification_number', readonly=False)

    def write(self, vals):
        for company in self:
            if 'l10n_sa_api_mode' in vals:
                if company.l10n_sa_api_mode == 'prod' and vals['l10n_sa_api_mode'] != 'prod':
                    raise UserError(_("You cannot change the ZATCA Submission Mode once it has been set to Production"))
                journals = self.env['account.journal'].search([('company_id', '=', company.id)])
                journals._l10n_sa_reset_certificates()
                journals.l10n_sa_latest_submission_hash = False
        return super().write(vals)

    def _get_company_address_field_names(self):
        """ Override to add ZATCA specific address fields """
        return super()._get_company_address_field_names() + \
            ['l10n_sa_edi_building_number', 'l10n_sa_edi_plot_identification']

    def _l10n_sa_edi_inverse_building_number(self):
        for company in self:
            company.partner_id.l10n_sa_edi_building_number = company.l10n_sa_edi_building_number

    def _l10n_sa_edi_inverse_plot_identification(self):
        for company in self:
            company.partner_id.l10n_sa_edi_plot_identification = company.l10n_sa_edi_plot_identification

    def _l10n_sa_get_csr_invoice_type(self):
        """
            Return the Invoice Type flag used in the CSR. 4-digit numerical input using 0 & 1 mapped to “TSCZ” where:
            -   0: False/Not supported, 1: True/Supported
            -   T: Tax Invoice (Standard), S: Simplified Invoice, C & Z will be used in the future and should
                always be 0
            For example: 1100 would mean the Solution will be generating Standard and Simplified invoices.
            We can assume Odoo-powered EGS solutions will always generate both Standard & Simplified invoices
        :return:
        """
        return '1100'

    def _l10n_sa_check_organization_unit(self):
        """
            Check company Organization Unit according to ZATCA specifications
            Standards:
                BR-KSA-39
                BR-KSA-40
            See https://zatca.gov.sa/ar/RulesRegulations/Taxes/Documents/20210528_ZATCA_Electronic_Invoice_XML_Implementation_Standard_vShared.pdf
        """
        self.ensure_one()
        if not self.vat:
            return False
        return len(self.vat) == 15 and bool(re.match(r'^3\d{13}3$', self.vat))

```

## File: models\res_config_settings.py

```python
from odoo import models, fields, api, _


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_sa_api_mode = fields.Selection(related='company_id.l10n_sa_api_mode', readonly=False)

    @api.depends('company_id')
    def _compute_company_informations(self):
        super()._compute_company_informations()
        for record in self:
            if self.company_id.country_code == 'SA':
                record.company_informations += _('\nBuilding Number: %s, Plot Identification: %s \nNeighborhood: %s') % (self.company_id.l10n_sa_edi_building_number, self.company_id.l10n_sa_edi_plot_identification, self.company_id.street2)

```

## File: models\res_partner.py

```python
from odoo import fields, models, api


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_sa_edi_building_number = fields.Char("Building Number")
    l10n_sa_edi_plot_identification = fields.Char("Plot Identification")

    l10n_sa_additional_identification_scheme = fields.Selection([
        ('TIN', 'Tax Identification Number'),
        ('CRN', 'Commercial Registration Number'),
        ('MOM', 'Momra License'),
        ('MLS', 'MLSD License'),
        ('700', '700 Number'),
        ('SAG', 'Sagia License'),
        ('NAT', 'National ID'),
        ('GCC', 'GCC ID'),
        ('IQA', 'Iqama Number'),
        ('PAS', 'Passport ID'),
        ('OTH', 'Other ID')
    ], default="OTH", string="Identification Scheme", help="Additional Identification scheme for Seller/Buyer")

    l10n_sa_additional_identification_number = fields.Char("Identification Number (SA)",
                                                           help="Additional Identification Number for Seller/Buyer")

    @api.model
    def _commercial_fields(self):
        return super()._commercial_fields() + ['l10n_sa_edi_building_number',
                                               'l10n_sa_edi_plot_identification',
                                               'l10n_sa_additional_identification_scheme',
                                               'l10n_sa_additional_identification_number']

    def _address_fields(self):
        return super()._address_fields() + ['l10n_sa_edi_building_number',
                                            'l10n_sa_edi_plot_identification']

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
from . import account_edi_format
from . import account_journal
from . import account_move
from . import account_tax
from . import res_partner
from . import res_company
from . import res_config_settings
from . import account_edi_xml_ubl_21_zatca
from . import ir_attachment

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
l10n_sa_edi_otp_wizard,l10n_sa_edi_otp_wizard,model_l10n_sa_edi_otp_wizard,account.group_account_invoice,1,1,1,0

```

## File: views\account_journal_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="view_account_journal_form" model="ir.ui.view">
            <field name="name">account.journal.form.l10n_sa_edi</field>
            <field name="model">account.journal</field>
            <field name="inherit_id" ref="account.view_account_journal_form"/>
            <field name="arch" type="xml">
                <xpath expr="//notebook" position="inside">
                    <field name="l10n_sa_csr" invisible="1"/>
                    <field name="l10n_sa_compliance_csid_json" invisible="1"/>
                    <field name="l10n_sa_production_csid_json" invisible="1"/>
                    <field name="l10n_sa_compliance_checks_passed" invisible="1"/>
                    <page name="zatca_einvoicing" string="ZATCA" attrs="{'invisible': ['|', ('country_code', '!=', 'SA'), ('type', '!=', 'sale')]}">
                        <group>
                            <group>
                                <field name="l10n_sa_serial_number"/>
                            </group>
                        </group>
                        <p groups="base.group_system">
                            <b>
                                In order to be able to submit Invoices to ZATCA, the following steps need to be completed:
                            </b>
                            <ol class="mt-2 mb-4">
                                <li>
                                    Set a Serial Number for your device
                                    <i class="fa fa-check text-success ms-1"
                                       attrs="{'invisible': [('l10n_sa_serial_number', '=', False)]}"/>
                                </li>
                                <li>
                                    Request a Compliance Certificate (CCSID)
                                    <i class="fa fa-check text-success ms-1" groups="base.group_system"
                                       attrs="{'invisible': [('l10n_sa_compliance_csid_json', '=', False)]}"/>
                                </li>
                                <li>
                                    Complete the Compliance Checks
                                    <i class="fa fa-check text-success ms-1"
                                       attrs="{'invisible': [('l10n_sa_compliance_checks_passed', '=', False)]}"/>
                                </li>
                                <li>
                                    Request a Production Certificate (PCSID)
                                    <i class="fa fa-check text-success ms-1" groups="base.group_system"
                                       attrs="{'invisible': [('l10n_sa_production_csid_json', '=', False)]}"/>
                                </li>
                            </ol>
                        </p>
                        <div class="alert alert-info d-flex justify-content-between align-items-center" role="alert" groups="base.group_system"
                             attrs="{'invisible':['|', ('l10n_sa_csr_errors', '!=', False), ('l10n_sa_compliance_csid_json', '!=', False)]}">
                            <p class="mb-0">
                                Onboard the Journal by completing each step
                            </p>
                            <button name="%(l10n_sa_edi_otp_wizard_act_window)d" type="action" icon="fa-key"
                                    class="btn-info ">
                                Onboard Journal
                            </button>
                        </div>
                        <div class="alert alert-danger d-flex flex-column align-items-end" role="alert" groups="base.group_system"
                             attrs="{'invisible':['|', '|', ('l10n_sa_csr_errors', '=', False), ('l10n_sa_compliance_csid_json', '!=', False), ('l10n_sa_production_csid_json', '!=', False)]}">
                            <div class="w-100">
                                <h4 role="alert" class="alert-heading">Journal could not be onboarded. Please make sure the Company VAT/Identification Number are correct.</h4>
                                <field name="l10n_sa_csr_errors" nolabel="1" readonly="1"/>
                                <hr/>
                            </div>
                            <button name="%(l10n_sa_edi_otp_wizard_act_window)d" type="action" icon="fa-key"
                                    class="btn-danger">
                                Onboard Journal
                            </button>
                        </div>
                        <div class="alert alert-info d-flex justify-content-between align-items-center" role="alert" groups="base.group_system"
                             attrs="{'invisible':['|', ('l10n_sa_compliance_checks_passed', '=', False), ('l10n_sa_production_csid_json', '=', False)]}">
                            <p class="mb-0">
                                The Production certificate is valid until
                                <field name="l10n_sa_production_csid_validity" readonly="1" nolabel="1"
                                       class="fw-bold"/>
                            </p>
                            <div>
                                <button name="%(l10n_sa_edi_otp_wizard_act_window)d" type="action" icon="fa-refresh"
                                        class="btn-info" context="{'default_l10n_sa_renewal': True}">
                                    Renew Production CSID
                                </button>
                                <button name="%(l10n_sa_edi_otp_wizard_act_window)d" type="action" icon="fa-refresh" class="btn-warning ms-2"
                                    confirm="Are you sure you wish to re-onboard the Journal?">
                                    Re-Onboard
                                </button>
                            </div>
                        </div>
                    </page>
                </xpath>
            </field>
        </record>

    </data>
</odoo>
```

## File: views\account_tax_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_tax_form" model="ir.ui.view">
        <field name="name">account.tax.form.zatca</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='country_id']" position="after">
                <field name="l10n_sa_is_retention" attrs="{'invisible': ['|', '|', ('type_tax_use', '!=', 'sale'), ('country_code', '!=', 'SA'), ('amount', '>=', 0)]}"/>
                <field name="l10n_sa_exemption_reason_code" attrs="{'invisible': ['|', '|', ('type_tax_use', '!=', 'sale'), ('country_code', '!=', 'SA'), ('amount', '!=', 0)]}"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <template id="arabic_english_invoice" inherit_id="l10n_sa.arabic_english_invoice">

            <!--    Add Currency Exchange rate if different currency than SAR    -->
            <xpath expr="//div[hasclass('clearfix')]" position="after">
                <table t-if="o.company_id.country_id.code == 'SA' and o.currency_id != o.company_id.currency_id"
                    id="sar_amounts" style="page-break-inside: avoid;" class="clearfix mx-auto mt-3 table-sm table-borderless">
                    <t t-set="curr_date" t-value="o.invoice_date or datetime.datetime.today()"></t>
                    <t t-set="sar_rate"
                    t-value="o.env['res.currency']._get_conversion_rate(o.currency_id, o.company_id.currency_id, o.company_id, curr_date)"/>
                    <tr>
                        <td class="w-25 text-start" dir="rtl"><strong>سعر الصرف</strong></td>
                        <td class="w-25 text-start" dir="rtl"><strong>الإجمالي الفرعي بالريال السعودي</strong></td>
                        <td class="w-25 text-start" dir="rtl"><strong>مبلغ ضريبة القيمة المضافة بالريال السعودي</strong></td>
                        <td class="w-25 text-start" dir="rtl"><strong>الإجمالي بالريال السعودي</strong></td>
                    </tr>
                    <tr>
                        <td class="w-25"><span class="fw-bold">Exchange Rate</span></td>
                        <td class="w-25"><span class="fw-bold">Subtotal</span></td>
                        <td class="w-25"><span class="fw-bold">VAT Amount</span></td>
                        <td class="w-25"><span class="fw-bold">Total</span></td>
                    </tr>
                    <tr>
                        <td><p class="m-0" t-esc="sar_rate" t-options='{"widget": "float", "precision": 5}'/></td>
                        <td><p class="m-0" t-esc="o.amount_untaxed_signed"
                            t-options='{"widget": "monetary", "display_currency": o.company_currency_id}'/></td>
                        <td><p class="m-0"
                            t-esc="o.currency_id._convert(o.amount_tax, o.company_id.currency_id, o.company_id, curr_date)"
                            t-options='{"widget": "monetary", "display_currency": o.company_currency_id}'/></td>
                        <td><p class="m-0" t-esc="o.amount_total_signed"
                            t-options='{"widget": "monetary", "display_currency": o.company_currency_id}'/></td>
                    </tr>
                </table>
            </xpath>

            <xpath expr="//t[@t-set='address']" position="inside">
                <div t-if="o.partner_id.l10n_sa_additional_identification_scheme and o.partner_id.l10n_sa_additional_identification_number" class="text-end mt0">
                        <span t-field="o.partner_id.l10n_sa_additional_identification_scheme"/>:
                        <span t-field="o.partner_id.l10n_sa_additional_identification_number"/>
                </div>
            </xpath>
            <xpath expr="//div[hasclass('col-4')]//span[@t-if=&quot;o.move_type == &apos;out_invoice&apos; and o.state == &apos;posted&apos;&quot;]" position="replace">
                <span t-if="o.move_type == 'out_invoice' and o.state == 'posted'">
                    <t t-if="o._l10n_sa_is_simplified()">
                        Simplified Tax Invoice
                    </t>
                    <t t-else="">
                        Tax Invoice
                    </t>
                </span>
            </xpath>
            <xpath expr="//div[hasclass('col-4')][3]//span[@t-if=&quot;o.move_type == &apos;out_invoice&apos; and o.state == &apos;posted&apos;&quot;]" position="replace">
                <span t-if="o.move_type == 'out_invoice' and o.state == 'posted'">
                    <t t-if="o._l10n_sa_is_simplified()">
                        فاتورة ضريبية مبسطة
                    </t>
                    <t t-else="">
                        فاتورة ضريبية
                    </t>
                </span>
            </xpath>

        </template>

    </data>
</odoo>

```

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_company_form" model="ir.ui.view">
            <field name="name">res.company.l10n_sa_edi.form</field>
            <field name="model">res.company</field>
            <field name="inherit_id" ref="base.view_company_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='country_id']" position="after">
                    <field name="l10n_sa_edi_building_number" placeholder="Building Number"
                           attrs="{'invisible': [('country_code', '!=', 'SA')]}"
                           class="o_address_building_number" options='{"no_open": True, "no_create": True}'/>
                    <field name="l10n_sa_edi_plot_identification" placeholder="Plot Identification"
                           attrs="{'invisible': [('country_code', '!=', 'SA')]}"
                           class="o_address_plot_identification" options='{"no_open": True, "no_create": True}'/>
                </xpath>
                <xpath expr="//field[@name='vat']" position="before">
                    <field name="l10n_sa_additional_identification_scheme" attrs="{'invisible': [('country_code', '!=', 'SA')]}"/>
                    <field name="l10n_sa_additional_identification_number" attrs="{'invisible': ['|', ('country_code', '!=', 'SA'), ('l10n_sa_additional_identification_scheme', '=', 'TIN')]}"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\res_config_settings_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.ui.view" id="res_config_settings_view_form">
        <field name="name">res.config.settings.view.form.inherit.l10n_sa_edi</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@data-key='account']/div" position="after">
                <field name="country_code" invisible="1"/>
                <h2 attrs="{'invisible':[('country_code', '!=', 'SA')]}">ZATCA E-Invoicing Settings</h2>
                <div class="row mt16 o_settings_container" name="saudi_zatca_edi" attrs="{'invisible':[('country_code', '!=', 'SA')]}">
                    <div class="col-12 o_setting_box">
                        <div class="o_setting_left_pane"/>
                        <div class="o_setting_right_pane">
                            <span class="o_form_label">ZATCA API Integration</span>
                            <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." aria-label="Values set here are company-specific." groups="base.group_multi_company" role="img"/>
                            <div class="text-muted">
                                You can select the API used for submissions down below. There are three modes available: Sandbox, Pre-Production and Production.
                                Once you have selected the correct API, you can start the Onboarding process by going to the Journals and checking the options under the ZATCA tab.
                            </div>
                            <div class="content-group">
                                 <div class="row mt8">
                                    <label for="l10n_sa_api_mode" class="col-2 o_light_label" string="API Mode"/>
                                    <field name="l10n_sa_api_mode" help="Set whether the system should use the Production API"/>
                                </div>
                            </div>
                            <div class="alert alert-warning mt8" role="alert">
                                <h4 class="alert-heading" role="alert">
                                    <i class="fa fa-warning me-2" /> Warning
                                </h4>
                                Once you change the submission mode to <strong>Production</strong>, you cannot change it anymore.
                                Be very careful, as any invoice submitted to ZATCA in Production mode will be accounted for
                                and might lead to <strong>Fines &amp; Penalties</strong>.
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="view_partner_form" model="ir.ui.view">
            <field name="name">res.partner.l10n_sa_edi.form</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="arch" type="xml">
                <field name="vat" position="before">
                    <field name="l10n_sa_additional_identification_scheme" attrs="{'invisible': [('country_code', '!=', 'SA')]}"/>
                    <field name="l10n_sa_additional_identification_number" attrs="{'invisible': ['|', ('country_code', '!=', 'SA'), ('l10n_sa_additional_identification_scheme', '=', 'TIN')]}"/>
                </field>
            </field>
        </record>

    </data>
</odoo>

```

## File: wizard\account_debit_note.py

```python
# -*- coding: utf-8 -*-
from odoo import models
from odoo.tools.translate import _
from odoo.exceptions import UserError


class AccountDebitNote(models.TransientModel):
    _inherit = 'account.debit.note'

    def create_debit(self):
        self.ensure_one()
        for move in self.move_ids:
            if move.journal_id.country_code == 'SA' and not self.reason:
                raise UserError(_("For debit notes issued in Saudi Arabia, you need to specify a Reason"))
        return super().create_debit()

```

## File: wizard\account_move_reversal.py

```python
# -*- coding: utf-8 -*-
from odoo import models
from odoo.tools.translate import _
from odoo.exceptions import UserError


class AccountMoveReversal(models.TransientModel):
    _inherit = 'account.move.reversal'

    def reverse_moves(self):
        self.ensure_one()
        for move in self.move_ids:
            if move.journal_id.country_code == 'SA' and not self.reason:
                raise UserError(_("For Credit/Debit notes issued in Saudi Arabia, you need to specify a Reason"))
        return super().reverse_moves()

```

## File: wizard\account_move_reversal_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_account_move_reversal_inherit_l10n_sa_edi" model="ir.ui.view">
            <field name="name">account.move.reversal.form.inherit.l10n_sa_edi</field>
            <field name="inherit_id" ref="account.view_account_move_reversal"/>
            <field name="model">account.move.reversal</field>
            <field name="arch" type="xml">
                <field name="reason" position="replace">
                    <field name="country_code" invisible="1"/>
                    <field name="reason" string="Reason" attrs="{'invisible': [('move_type', '=', 'entry'), ('country_code', '!=', 'SA')], 'required': [('country_code', '=', 'SA')]}"/>
                </field>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\l10n_sa_edi_otp_wizard.py

```python
from odoo import fields, models, _, api
from odoo.exceptions import UserError


class RequestZATCAOtp(models.TransientModel):
    _name = 'l10n_sa_edi.otp.wizard'
    _description = 'Request ZATCA OTP'

    l10n_sa_renewal = fields.Boolean("PCSID Renewal",
                                     help="Used to decide whether we should call the PCSID renewal API or the CCSID API",
                                     default=False)
    l10n_sa_otp = fields.Char("OTP", copy=False, help="OTP required to get a CCSID. Can only be acquired through "
                                                      "the Fatoora portal.")
    journal_id = fields.Many2one('account.journal', default=lambda self: self.env.context.get('active_id'), required=True)

    @api.model
    def default_get(self, fields):
        res = super().default_get(fields)
        if self.env.company.l10n_sa_api_mode == 'sandbox':
            res['l10n_sa_otp'] = '123456' if self.l10n_sa_renewal else '123345'
        return res

    def validate(self):
        if not self.l10n_sa_otp:
            raise UserError(_("You need to provide an OTP to be able to request a CCSID"))
        if self.l10n_sa_renewal:
            return self.journal_id._l10n_sa_get_production_CSID(self.l10n_sa_otp)
        self.journal_id._l10n_sa_api_onboard_journal(self.l10n_sa_otp)

```

## File: wizard\l10n_sa_edi_otp_wizard.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="l10n_sa_edi_otp_wizard_view_form" model="ir.ui.view">
        <field name="name">l10n_sa_edi.otp.wizard.form</field>
        <field name="model">l10n_sa_edi.otp.wizard</field>
        <field name="arch" type="xml">
            <form string="Use an OTP to request for a CSID">
                Please, set the OTP you received from ZATCA in the input below then validate.
                <group>
                    <field name="journal_id" invisible="1"/>
                    <field name="l10n_sa_renewal" invisible="1"/>
                    <field name="l10n_sa_otp"/>
                </group>
                <footer>
                    <button string="Request" type="object" name="validate" class="btn btn-primary" data-hotkey="q"/>
                    <button string="Cancel" special="cancel" data-hotkey="z" class="btn btn-secondary"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="l10n_sa_edi_otp_wizard_act_window" model="ir.actions.act_window">
        <field name="name">Request a CSID</field>
        <field name="res_model">l10n_sa_edi.otp.wizard</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>

</odoo>

```

## File: wizard\__init__.py

```python
from . import account_move_reversal
from . import account_debit_note
from . import l10n_sa_edi_otp_wizard

```

