# Odoo Module: account_edi_ubl_cii

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': "Import/Export electronic invoices with UBL/CII",
    'version': '1.0',
    'category': 'Accounting/Accounting',
    'description': """
Electronic invoicing module
===========================

Allows to export and import formats: E-FFF, UBL Bis 3, EHF3, Factur-X (CII), XRechnung (UBL).
When generating the PDF on the invoice, the PDF will be embedded inside the xml for all UBL formats. This allows the
receiver to retrieve the PDF with only the xml file. Note that **EHF3 is fully implemented by UBL Bis 3** (`reference 
<https://anskaffelser.dev/postaward/g3/spec/current/billing-3.0/norway/#_implementation>`_).

The formats can be chosen from the journal (Journal > Advanced Settings) linked to the invoice. 

Note that E-FFF and XRechnung (UBL) are only available for Belgian and German companies, 
respectively. UBL Bis 3 is only available for companies which country is present in the `EAS list 
<https://docs.peppol.eu/poacc/billing/3.0/codelist/eas/>`_.

Note also that you need to activate PDF A in order to be able to submit a Factur-X pdf on Chorus Pro: 
go to Settings > Technical (debug mode) > System Parameters > select/create one with Key: edi.use_pdfa, Value: true. 
With this setting, Chorus Pro will automatically detect the "PDF/A-3 (Factur-X)" format.
    """,
    'depends': ['account_edi'],
    'data': [
        'data/account_edi_data.xml',
        'data/cii_22_templates.xml',
        'data/ubl_20_templates.xml',
        'data/ubl_21_templates.xml',
    ],
    'installable': True,
    'application': False,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: data\account_edi_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="ubl_bis3" model="account.edi.format">
            <field name="name">Peppol BIS Billing 3.0</field>
            <field name="code">ubl_bis3</field>
        </record>

        <record id="ubl_de" model="account.edi.format">
            <field name="name">XRechnung UBL (Germany)</field>
            <field name="code">ubl_de</field>
        </record>
    </data>
</odoo>

```

## File: data\cii_22_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="account_invoice_line_facturx_export_22">
            <t t-set="line" t-value="line_vals['line']"/>
            <t  xmlns:ram="urn:un:unece:uncefact:data:standard:ReusableAggregateBusinessInformationEntity:100"
                xmlns:rsm="urn:un:unece:uncefact:data:standard:CrossIndustryInvoice:100"
                xmlns:udt="urn:un:unece:uncefact:data:standard:UnqualifiedDataType:100">

                <ram:IncludedSupplyChainTradeLineItem>
                    <!-- Line number. -->
                    <ram:AssociatedDocumentLineDocument>
                        <ram:LineID t-esc="line_vals.get('index')"/>
                    </ram:AssociatedDocumentLineDocument>

                    <!-- Product. -->
                    <ram:SpecifiedTradeProduct>
                        <ram:GlobalID
                            schemeID="0160"
                            t-if="line.product_id and line.product_id.barcode"
                            t-esc="line.product_id.barcode"/>
                        <ram:SellerAssignedID
                            t-if="line.product_id and line.product_id.default_code"
                            t-esc="line.product_id.default_code"/>
                        <ram:Name t-esc="line.name"/>
                    </ram:SpecifiedTradeProduct>

                    <!-- Amounts. -->
                    <ram:SpecifiedLineTradeAgreement>
                        <!-- Line information: unit_price
                        NB: the gross_price_unit should be the price tax excluded !
                        if price_unit = 100 and tax 10% price_include -> then the gross_price_unit is 91
                        -->
                        <ram:GrossPriceProductTradePrice>
                            <ram:ChargeAmount t-esc="format_monetary(line_vals['gross_price_total_unit'], 2)"/>
                            <!-- Discount. -->
                            <ram:AppliedTradeAllowanceCharge t-if="line.discount">
                                <ram:ChargeIndicator>
                                    <udt:Indicator>false</udt:Indicator>
                                </ram:ChargeIndicator>
                                <ram:ActualAmount t-esc="format_monetary(line_vals['price_discount_unit'], 2)"/>
                            </ram:AppliedTradeAllowanceCharge>
                        </ram:GrossPriceProductTradePrice>
                        <!-- Line unit price, with discount applied -->
                        <ram:NetPriceProductTradePrice>
                            <ram:ChargeAmount
                                t-esc="format_monetary(line_vals['price_subtotal_unit'], 2)"/>
                        </ram:NetPriceProductTradePrice>
                    </ram:SpecifiedLineTradeAgreement>

                    <!-- Quantity. -->
                    <ram:SpecifiedLineTradeDelivery>
                        <ram:BilledQuantity
                                t-att-unitCode="line_vals.get('unece_uom_code')"
                                t-esc="line.quantity"/> <!-- /!\ The quantity is the line.quantity since we keep the unece_uom_code ! -->
                    </ram:SpecifiedLineTradeDelivery>

                    <ram:SpecifiedLineTradeSettlement>
                        <t t-foreach="tax_details['invoice_line_tax_details'][line]['tax_details'].values()"
                           t-as="tax_detail_vals">
                            <ram:ApplicableTradeTax t-if="tax_detail_vals['amount_type'] == 'percent'">
                                <ram:TypeCode>VAT</ram:TypeCode>
                                <ram:CategoryCode t-esc="tax_detail_vals['tax_category_code']"/>
                                <ram:RateApplicablePercent t-esc="tax_detail_vals['amount']"/>
                            </ram:ApplicableTradeTax>
                        </t>

                        <!-- Allowance/Charge on the line -->
                        <t t-foreach="line_vals.get('allowance_charge_vals_list')" t-as="allowance_charge_vals">
                            <ram:SpecifiedTradeAllowanceCharge>
                                <ram:ChargeIndicator>
                                    <udt:Indicator t-esc="allowance_charge_vals['indicator']"/>
                                </ram:ChargeIndicator>
                                <ram:ActualAmount t-esc="format_monetary(allowance_charge_vals['amount'], 2)"/>
                                <ram:ReasonCode t-esc="allowance_charge_vals['reason_code']"/>
                                <ram:Reason t-esc="allowance_charge_vals['reason']"/>
                            </ram:SpecifiedTradeAllowanceCharge>
                        </t>

                        <!-- Subtotal. -->
                        <ram:SpecifiedTradeSettlementLineMonetarySummation>
                            <ram:LineTotalAmount t-esc="format_monetary(line_vals['line_total_amount'], 2)"/>
                        </ram:SpecifiedTradeSettlementLineMonetarySummation>

                    </ram:SpecifiedLineTradeSettlement>
                </ram:IncludedSupplyChainTradeLineItem>
            </t>
        </template>

        <template id="account_invoice_partner_facturx_export_22">
            <t  xmlns:ram="urn:un:unece:uncefact:data:standard:ReusableAggregateBusinessInformationEntity:100"
                xmlns:rsm="urn:un:unece:uncefact:data:standard:CrossIndustryInvoice:100"
                xmlns:udt="urn:un:unece:uncefact:data:standard:UnqualifiedDataType:100">
                <!-- Contact. -->
                <ram:Name t-esc="partner.name"/>
                <ram:SpecifiedLegalOrganization t-if="specified_legal_organization_val">
                    <ram:ID t-att-schemeID="str('0002')"
                            t-esc="specified_legal_organization_val"/>
                </ram:SpecifiedLegalOrganization>

                <ram:DefinedTradeContact t-if="not hide_dtc">
                    <ram:PersonName t-esc="partner.name"/>
                    <ram:TelephoneUniversalCommunication t-if="partner.phone or partner.mobile">
                        <ram:CompleteNumber t-esc="partner.phone or partner.mobile"/>
                    </ram:TelephoneUniversalCommunication>
                    <ram:EmailURIUniversalCommunication t-if="partner.email">
                        <ram:URIID schemeID='SMTP' t-esc="partner.email"/>
                    </ram:EmailURIUniversalCommunication>
                </ram:DefinedTradeContact>

                <!-- Address. -->
                <t t-call="account_edi_ubl_cii.account_invoice_address_facturx_export_22"/>
            </t>
        </template>

        <template id="account_invoice_address_facturx_export_22">
            <t  xmlns:ram="urn:un:unece:uncefact:data:standard:ReusableAggregateBusinessInformationEntity:100"
                xmlns:rsm="urn:un:unece:uncefact:data:standard:CrossIndustryInvoice:100"
                xmlns:udt="urn:un:unece:uncefact:data:standard:UnqualifiedDataType:100">
                <!-- Address. -->
                <ram:PostalTradeAddress>
                    <ram:PostcodeCode t-esc="partner.zip"/>
                    <ram:LineOne t-esc="partner.street"/>
                    <ram:LineTwo t-esc="partner.street2"/>
                    <ram:CityName t-esc="partner.city"/>
                    <ram:CountryID t-esc="partner.country_id.code"/>
                </ram:PostalTradeAddress>
            </t>
        </template>

        <template id="account_invoice_facturx_export_22">
            <t t-set="currency" t-value="record.currency_id or record.company_currency_id"/>
            <rsm:CrossIndustryInvoice
                xmlns:ram="urn:un:unece:uncefact:data:standard:ReusableAggregateBusinessInformationEntity:100"
                xmlns:rsm="urn:un:unece:uncefact:data:standard:CrossIndustryInvoice:100"
                xmlns:udt="urn:un:unece:uncefact:data:standard:UnqualifiedDataType:100">
                <!-- Factur-x level:
                    * minimum or basicwl:   urn:factur-x.eu:1p0...
                    * basic:                urn:cen.eu:en16931:2017:compliant:factur-x.eu:1p0:basic
                    * en16931:              urn:cen.eu:en16931:2017
                -->
                <rsm:ExchangedDocumentContext>
                    <ram:GuidelineSpecifiedDocumentContextParameter>
                        <ram:ID t-esc="document_context_id"/>
                    </ram:GuidelineSpecifiedDocumentContextParameter>
                </rsm:ExchangedDocumentContext>

                <!-- Document Headers. -->
                <rsm:ExchangedDocument>
                    <ram:ID t-esc="ExchangedDocument_vals.get('id')"/>
                    <ram:TypeCode t-esc="ExchangedDocument_vals.get('type_code')"/>
                    <ram:IssueDateTime t-if="ExchangedDocument_vals.get('issue_date_time')">
                        <udt:DateTimeString format="102" t-esc="format_date(ExchangedDocument_vals.get('issue_date_time'))"/>
                    </ram:IssueDateTime>
                    <ram:IncludedNote>
                        <ram:Content t-esc="ExchangedDocument_vals.get('included_note')"/>
                    </ram:IncludedNote>
                </rsm:ExchangedDocument>

                <rsm:SupplyChainTradeTransaction>
                    <!-- Invoice lines. -->
                    <t t-foreach="invoice_line_vals_list" t-as="line_vals">
                        <t t-call="account_edi_ubl_cii.account_invoice_line_facturx_export_22"/>
                    </t>

                    <!-- Partners. -->
                    <ram:ApplicableHeaderTradeAgreement>
                        <!-- "Service Exécutant" (Used by Chorus Pro) -->
                        <ram:BuyerReference t-esc="buyer_reference"/>
                        <!-- Seller. -->
                        <ram:SellerTradeParty>
                            <!-- Address. -->
                            <t t-call="account_edi_ubl_cii.account_invoice_partner_facturx_export_22">
                                <t t-set="partner" t-value="record.company_id.partner_id"/>
                                <t t-set="specified_legal_organization_val" t-value="seller_specified_legal_organization"/>
                            </t>

                            <!-- VAT. -->
                            <ram:SpecifiedTaxRegistration t-if="record.company_id.vat">
                                <ram:ID schemeID="VA" t-esc="record.company_id.vat"/>
                            </ram:SpecifiedTaxRegistration>
                        </ram:SellerTradeParty>

                        <!-- Customer. -->
                        <ram:BuyerTradeParty>
                            <!-- Address. -->
                            <t t-call="account_edi_ubl_cii.account_invoice_partner_facturx_export_22">
                                <t t-set="partner" t-value="record.commercial_partner_id"/>
                                <t t-set="specified_legal_organization_val" t-value="buyer_specified_legal_organization"/>
                            </t>

                            <!-- VAT. -->
                            <ram:SpecifiedTaxRegistration t-if="record.commercial_partner_id.vat">
                                <ram:ID schemeID="VA" t-esc="record.commercial_partner_id.vat"/>
                            </ram:SpecifiedTaxRegistration>
                        </ram:BuyerTradeParty>

                        <!-- Reference. -->
                        <ram:BuyerOrderReferencedDocument>
                            <!-- "Engagement Juridique" (used by Chorus Pro) -->
                            <ram:IssuerAssignedID t-esc="purchase_order_reference"/>
                        </ram:BuyerOrderReferencedDocument>
                        <ram:ContractReferencedDocument>
                            <!-- "Numéro de Marché" (Used by Chorus Pro) -->
                            <ram:IssuerAssignedID t-esc="contract_reference"/>
                        </ram:ContractReferencedDocument>
                    </ram:ApplicableHeaderTradeAgreement>

                    <!-- Delivery -->
                    <ram:ApplicableHeaderTradeDelivery>
                        <ram:ShipToTradeParty>
                            <t t-call="account_edi_ubl_cii.account_invoice_partner_facturx_export_22">
                                <t t-set="partner"
                                   t-value="ship_to_trade_party"/>
                                <t t-set="hide_dtc" t-value="document_context_id == 'urn:cen.eu:en16931:2017#compliant#urn:xoev-de:kosit:standard:xrechnung_2.2'"/>
                            </t>
                        </ram:ShipToTradeParty>

                        <ram:ActualDeliverySupplyChainEvent t-if="scheduled_delivery_time">
                            <ram:OccurrenceDateTime>
                                <udt:DateTimeString t-att-format="102"
                                                    t-esc="format_date(scheduled_delivery_time)"/>
                            </ram:OccurrenceDateTime>
                        </ram:ActualDeliverySupplyChainEvent>
                    </ram:ApplicableHeaderTradeDelivery>

                    <!-- Taxes. -->
                    <ram:ApplicableHeaderTradeSettlement>

                        <ram:PaymentReference t-esc="record.payment_reference"/>
                        <ram:InvoiceCurrencyCode t-esc="currency.name"/>

                        <!-- Bank account. -->
                        <ram:SpecifiedTradeSettlementPaymentMeans t-if="record.partner_bank_id.sanitized_acc_number">
                            <ram:TypeCode>42</ram:TypeCode>
                            <ram:PayeePartyCreditorFinancialAccount>
                                <ram:IBANID
                                    t-if="record.partner_bank_id.sanitized_acc_number == 'iban'"
                                    t-esc="record.partner_bank_id.sanitized_acc_number"/>
                                <ram:ProprietaryID
                                    t-if="record.partner_bank_id.sanitized_acc_number != 'iban'"
                                    t-esc="record.partner_bank_id.sanitized_acc_number"/>
                            </ram:PayeePartyCreditorFinancialAccount>
                        </ram:SpecifiedTradeSettlementPaymentMeans>

                        <!-- Tax Summary. -->
                        <t t-foreach="tax_details.get('tax_details', {}).values()" t-as="tax_detail_vals">
                            <ram:ApplicableTradeTax>
                                <ram:CalculatedAmount
                                    t-esc="format_monetary(tax_detail_vals.get('calculated_amount'), 2)"/>
                                <ram:TypeCode>VAT</ram:TypeCode>
                                <ram:ExemptionReason t-esc="tax_detail_vals.get('tax_exemption_reason')"/>
                                <ram:BasisAmount
                                    t-esc="format_monetary(balance_multiplicator * tax_detail_vals['base_amount_currency'], 2)"/>
                                <ram:CategoryCode t-esc="tax_detail_vals['tax_category_code']"/>
                                <ram:ExemptionReasonCode t-esc="tax_detail_vals['tax_exemption_reason_code']"/>
                                <!-- 5 = Tax is exigible on the date on the invoice -->
                                <ram:DueDateTypeCode>5</ram:DueDateTypeCode>
                                <ram:RateApplicablePercent
                                    t-if="tax_detail_vals['amount_type'] == 'percent'"
                                    t-esc="tax_detail_vals['amount']"/>
                            </ram:ApplicableTradeTax>
                        </t>

                        <!-- Billing Period -->
                        <ram:BillingSpecifiedPeriod t-if="billing_start and billing_end">
                            <ram:StartDateTime>
                                <udt:DateTimeString format="102" t-esc="format_date(billing_start)"/>
                            </ram:StartDateTime>
                            <ram:EndDateTime>
                                <udt:DateTimeString format="102" t-esc="format_date(billing_end)"/>
                            </ram:EndDateTime>
                        </ram:BillingSpecifiedPeriod>

                        <!-- Payment Term. -->
                        <ram:SpecifiedTradePaymentTerms>
                            <ram:Description t-if="record.invoice_payment_term_id" t-esc="record.invoice_payment_term_id.name"/>
                            <ram:DueDateDateTime t-if="record.invoice_date_due">
                                <udt:DateTimeString format="102" t-esc="format_date(record.invoice_date_due)"/>
                            </ram:DueDateDateTime>
                        </ram:SpecifiedTradePaymentTerms>

                        <!-- Summary. -->
                        <ram:SpecifiedTradeSettlementHeaderMonetarySummation>
                            <ram:LineTotalAmount
                                t-esc="format_monetary(tax_basis_total_amount, 2)"/>
                            <ram:TaxBasisTotalAmount
                                t-esc="format_monetary(tax_basis_total_amount, 2)"/>
                            <ram:TaxTotalAmount
                                t-att-currencyID="currency.name"
                                t-esc="format_monetary(tax_total_amount, 2)"/>
                            <ram:GrandTotalAmount
                                t-esc="format_monetary(record.amount_total, 2)"/>
                            <ram:TotalPrepaidAmount
                                t-esc="format_monetary(record.amount_total - record.amount_residual, 2)"/>
                            <ram:DuePayableAmount
                                t-esc="format_monetary(record.amount_residual, 2)"/>
                        </ram:SpecifiedTradeSettlementHeaderMonetarySummation>
                    </ram:ApplicableHeaderTradeSettlement>
                </rsm:SupplyChainTradeTransaction>
            </rsm:CrossIndustryInvoice>
        </template>

        <template id="account_invoice_pdfa_3_facturx_metadata">
            <x:xmpmeta xmlns:x="adobe:ns:meta/">
                <rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
                    <rdf:Description xmlns:pdfaid="http://www.aiim.org/pdfa/ns/id/" rdf:about="">
                        <pdfaid:part>3</pdfaid:part>
                        <pdfaid:conformance>B</pdfaid:conformance>
                    </rdf:Description>
                    <rdf:Description xmlns:dc="http://purl.org/dc/elements/1.1/" rdf:about="">
                        <dc:title>
                            <rdf:Alt>
                                <rdf:li t-att="{'xml:lang': 'x-default'}" t-esc="title"/>
                            </rdf:Alt>
                        </dc:title>
                        <dc:creator>
                            <rdf:Seq>
                                <rdf:li>Odoo</rdf:li>
                            </rdf:Seq>
                        </dc:creator>
                        <dc:description>
                            <rdf:Alt>
                                <rdf:li t-att="{'xml:lang': 'x-default'}">Invoice generated by Odoo</rdf:li>
                            </rdf:Alt>
                        </dc:description>
                    </rdf:Description>
                    <rdf:Description xmlns:pdf="http://ns.adobe.com/pdf/1.3/" rdf:about="">
                        <pdf:Producer>Odoo</pdf:Producer>
                    </rdf:Description>
                    <rdf:Description xmlns:xmp="http://ns.adobe.com/xap/1.0/" rdf:about="">
                        <xmp:CreatorTool>Odoo</xmp:CreatorTool>
                        <xmp:CreateDate t-esc="date"/>
                        <xmp:ModifyDate t-esc="date"/>
                    </rdf:Description>
                    <rdf:Description xmlns:pdfaExtension="http://www.aiim.org/pdfa/ns/extension/"
                                     xmlns:pdfaSchema="http://www.aiim.org/pdfa/ns/schema#"
                                     xmlns:pdfaProperty="http://www.aiim.org/pdfa/ns/property#" rdf:about="">
                        <pdfaExtension:schemas>
                            <rdf:Bag>
                                <rdf:li rdf:parseType="Resource">
                                    <pdfaSchema:schema>Factur-X PDFA Extension Schema</pdfaSchema:schema>
                                    <pdfaSchema:namespaceURI>urn:factur-x:pdfa:CrossIndustryDocument:invoice:1p0#</pdfaSchema:namespaceURI>
                                    <pdfaSchema:prefix>fx</pdfaSchema:prefix>
                                    <pdfaSchema:property>
                                        <rdf:Seq>
                                            <rdf:li rdf:parseType="Resource">
                                                <pdfaProperty:name>DocumentFileName</pdfaProperty:name>
                                                <pdfaProperty:valueType>Text</pdfaProperty:valueType>
                                                <pdfaProperty:category>external</pdfaProperty:category>
                                                <pdfaProperty:description>name of the embedded XML invoice file</pdfaProperty:description>
                                            </rdf:li>
                                            <rdf:li rdf:parseType="Resource">
                                                <pdfaProperty:name>DocumentType</pdfaProperty:name>
                                                <pdfaProperty:valueType>Text</pdfaProperty:valueType>
                                                <pdfaProperty:category>external</pdfaProperty:category>
                                                <pdfaProperty:description>INVOICE</pdfaProperty:description>
                                            </rdf:li>
                                            <rdf:li rdf:parseType="Resource">
                                                <pdfaProperty:name>Version</pdfaProperty:name>
                                                <pdfaProperty:valueType>Text</pdfaProperty:valueType>
                                                <pdfaProperty:category>external</pdfaProperty:category>
                                                <pdfaProperty:description>The actual version of the Factur-X XML schema</pdfaProperty:description>
                                            </rdf:li>
                                            <rdf:li rdf:parseType="Resource">
                                                <pdfaProperty:name>ConformanceLevel</pdfaProperty:name>
                                                <pdfaProperty:valueType>Text</pdfaProperty:valueType>
                                                <pdfaProperty:category>external</pdfaProperty:category>
                                                <pdfaProperty:description>The conformance level of the embedded Factur-X data</pdfaProperty:description>
                                            </rdf:li>
                                        </rdf:Seq>
                                    </pdfaSchema:property>
                                </rdf:li>
                            </rdf:Bag>
                        </pdfaExtension:schemas>
                    </rdf:Description>
                    <rdf:Description xmlns:fx="urn:factur-x:pdfa:CrossIndustryDocument:invoice:1p0#" rdf:about="">
                        <fx:ConformanceLevel>EN 16931</fx:ConformanceLevel>
                        <fx:DocumentFileName>factur-x.xml</fx:DocumentFileName>
                        <fx:DocumentType>INVOICE</fx:DocumentType>
                        <fx:Version>1.0</fx:Version>
                    </rdf:Description>
                </rdf:RDF>
            </x:xmpmeta>
        </template>

    </data>
</odoo>

```

## File: data\ubl_20_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="ubl_20_AddressType">
        <t xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
           xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
           xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:StreetName t-esc="vals.get('street_name')"/>
            <cbc:AdditionalStreetName t-esc="vals.get('additional_street_name')"/>
            <cbc:CityName t-esc="vals.get('city_name')"/>
            <cbc:PostalZone t-esc="vals.get('postal_zone')"/>
            <cbc:CountrySubentity t-esc="vals.get('country_subentity')"/>
            <cbc:CountrySubentityCode t-esc="vals.get('country_subentity_code')"/>
            <cac:Country>
                <t t-set="country_vals" t-value="vals.get('country_vals', {})"/>
                <cbc:IdentificationCode
                    t-att="country_vals.get('identification_code_attrs', {})"
                    t-esc="country_vals.get('identification_code')"/>
                <cbc:Name t-esc="country_vals.get('name')"/>
            </cac:Country>
        </t>
    </template>

    <template id="ubl_20_PartyType">
        <t xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
           xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
           xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <t t-set="partner" t-value="vals.get('partner') or builder.env['res.partner']"/>
            <cbc:EndpointID
                    t-att="vals.get('endpoint_id_attrs', {})"
                    t-esc="vals.get('endpoint_id')"/>
            <t t-foreach="vals.get('party_identification_vals', [])" t-as="party_vals">
                <cac:PartyIdentification>
                    <cbc:ID
                        t-att="party_vals.get('id_attrs', {})"
                        t-esc="party_vals.get('id')"/>
                </cac:PartyIdentification>
            </t>
            <t t-foreach="vals.get('party_name_vals', [])" t-as="party_vals">
                <cac:PartyName>
                    <cbc:Name t-esc="party_vals.get('name')"/>
                </cac:PartyName>
            </t>
            <cac:PostalAddress>
                <t t-call="{{AddressType_template}}">
                    <t t-set="vals" t-value="vals.get('postal_address_vals', {})"/>
                </t>
            </cac:PostalAddress>
            <t t-foreach="vals.get('party_tax_scheme_vals', [])" t-as="foreach_vals">
                <cac:PartyTaxScheme>
                    <cbc:RegistrationName t-esc="foreach_vals.get('registration_name')"/>
                    <cbc:CompanyID
                            t-att="foreach_vals.get('company_id_attrs', {})"
                            t-esc="foreach_vals.get('company_id')"/>
                    <cac:RegistrationAddress>
                        <t t-call="{{AddressType_template}}">
                            <t t-set="vals" t-value="foreach_vals.get('registration_address_vals', {})"/>
                        </t>
                    </cac:RegistrationAddress>
                    <cac:TaxScheme>
                        <cbc:ID t-esc="foreach_vals.get('tax_scheme_id')"/>
                    </cac:TaxScheme>
                </cac:PartyTaxScheme>
            </t>

            <t t-foreach="vals.get('party_legal_entity_vals', [])" t-as="foreach_vals">
                <cac:PartyLegalEntity>
                    <cbc:RegistrationName t-esc="foreach_vals.get('registration_name')"/>
                    <cbc:CompanyID
                            t-att="foreach_vals.get('company_id_attrs', {})"
                            t-esc="foreach_vals.get('company_id')"/>
                    <cac:RegistrationAddress>
                        <t t-call="{{AddressType_template}}">
                            <t t-set="vals" t-value="foreach_vals.get('registration_address_vals', {})"/>
                        </t>
                    </cac:RegistrationAddress>
                </cac:PartyLegalEntity>
            </t>
            <cac:Contact>
                <t t-set="contact_vals" t-value="vals.get('contact_vals', {})"/>
                <cbc:ID t-esc="contact_vals.get('id')"/>
                <cbc:Name t-esc="contact_vals.get('name')"/>
                <cbc:Telephone t-esc="contact_vals.get('telephone')"/>
                <cbc:ElectronicMail t-esc="contact_vals.get('electronic_mail')"/>
            </cac:Contact>
        </t>
    </template>

    <template id="ubl_20_PaymentMeansType">
        <t xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
           xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
           xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:PaymentMeansCode
                    t-att="vals.get('payment_means_code_attrs', {})"
                    t-esc="vals.get('payment_means_code')"/>
            <cbc:PaymentDueDate t-esc="vals.get('payment_due_date')"/>
            <cbc:InstructionID t-esc="vals.get('instruction_id')"/>
            <t t-foreach="vals.get('payment_id_vals', [])" t-as="payment_id">
                <cbc:PaymentID t-esc="payment_id"/>
            </t>
            <cac:PayeeFinancialAccount>
                <t t-set="payee_financial_account_vals" t-value="vals.get('payee_financial_account_vals', {})"/>
                <cbc:ID
                    t-att="payee_financial_account_vals.get('id_attrs', {})"
                    t-esc="payee_financial_account_vals.get('id')"/>
                <cac:FinancialInstitutionBranch>
                    <t t-set="financial_institution_branch_vals" t-value="payee_financial_account_vals.get('financial_institution_branch_vals', {})"/>
                    <cbc:ID
                        t-att="financial_institution_branch_vals.get('id_attrs', {})"
                        t-esc="financial_institution_branch_vals.get('id')"/>
                    <cac:FinancialInstitution>
                        <t t-set="financial_institution_vals" t-value="financial_institution_branch_vals.get('financial_institution_vals', {})"/>
                        <cbc:ID
                            t-att="financial_institution_vals.get('id_attrs', {})"
                            t-esc="financial_institution_vals.get('id')"/>
                        <cbc:Name t-esc="financial_institution_vals.get('name')"/>
                        <cac:Address>
                            <t t-call="{{AddressType_template}}">
                                <t t-set="vals" t-value="financial_institution_vals.get('address_vals', {})"/>
                            </t>
                        </cac:Address>
                    </cac:FinancialInstitution>
                </cac:FinancialInstitutionBranch>
            </cac:PayeeFinancialAccount>
        </t>
    </template>

    <template id="ubl_20_TaxCategoryType">
        <t xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
           xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
           xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:ID
                t-att="vals.get('id_attrs', {})"
                t-esc="vals.get('id')"/>
            <cbc:Name t-esc="vals.get('name')"/>
            <cbc:Percent t-esc="vals.get('percent')"/>
            <cbc:TaxExemptionReasonCode
                    t-esc="vals.get('tax_exemption_reason_code')"/>
            <cbc:TaxExemptionReason
                    t-esc="vals.get('tax_exemption_reason')"/>
            <cac:TaxScheme>
                <cbc:ID>VAT</cbc:ID>
            </cac:TaxScheme>
        </t>
    </template>

    <template id="ubl_20_TaxTotalType">
        <t xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
           xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
           xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:TaxAmount
                    t-att-currencyID="vals['currency'].name"
                    t-esc="format_float(vals.get('tax_amount'), vals.get('currency_dp'))"/>
            <t t-foreach="vals.get('tax_subtotal_vals', [])" t-as="foreach_vals">
                <cac:TaxSubtotal>
                    <cbc:TaxableAmount
                        t-att-currencyID="foreach_vals['currency'].name"
                        t-esc="format_float(foreach_vals.get('taxable_amount'), foreach_vals.get('currency_dp'))"/>
                    <cbc:TaxAmount
                        t-att-currencyID="foreach_vals['currency'].name"
                        t-esc="format_float(foreach_vals.get('tax_amount'), foreach_vals.get('currency_dp'))"/>
                    <cbc:Percent t-esc="foreach_vals.get('percent')"/>
                    <cac:TaxCategory>
                        <t t-call="{{TaxCategoryType_template}}">
                            <t t-set="vals" t-value="foreach_vals.get('tax_category_vals', {})"/>
                        </t>
                    </cac:TaxCategory>
                </cac:TaxSubtotal>
            </t>
        </t>
    </template>

    <template id="ubl_20_AllowanceChargeType">
        <t xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
           xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
           xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:ChargeIndicator t-esc="vals.get('charge_indicator')"/>
            <cbc:AllowanceChargeReasonCode t-esc="vals.get('allowance_charge_reason_code')"/>
            <cbc:AllowanceChargeReason t-esc="vals.get('allowance_charge_reason')"/>
            <cbc:MultiplierFactorNumeric t-esc="vals.get('multiplier_factor')"/>
            <cbc:Amount
                    t-att-currencyID="vals.get('currency_name')"
                    t-esc="format_float(vals.get('amount'), vals.get('currency_dp'))"/>
            <t t-foreach="vals.get('tax_category_vals', [])" t-as="foreach_vals">
                <cac:TaxCategory>
                    <t t-call="{{TaxCategoryType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:TaxCategory>
            </t>
        </t>
    </template>

    <template id="ubl_20_InvoiceLineType">
        <t xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
           xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
           xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:ID t-esc="vals.get('id')"/>
            <cbc:Note t-esc="vals.get('note')"/>
            <cbc:InvoicedQuantity
                    t-if="invoice.move_type == 'out_invoice'"
                    t-att="vals.get('invoiced_quantity_attrs', {})"
                    t-esc="vals.get('invoiced_quantity')"/>
            <cbc:CreditedQuantity
                    t-if="invoice.move_type == 'out_refund'"
                    t-att="vals.get('invoiced_quantity_attrs', {})"
                    t-esc="vals.get('invoiced_quantity')"/>
            <cbc:LineExtensionAmount
                    t-att-currencyID="vals['currency'].name"
                    t-esc="format_float(vals.get('line_extension_amount'), vals.get('currency_dp'))"/>
            <!-- AllowanceCharge is only present for InvoiceLine, not for CreditNoteLine in UBL 2.0
            (they are both present in UBL 2.1 and next)
             http://www.datypic.com/sc/ubl20/e-cac_CreditNoteLine.html
             http://www.datypic.com/sc/ubl20/e-cac_InvoiceLine.html
             -->
            <t t-if="invoice.move_type == 'out_invoice'">
                <t t-foreach="vals.get('allowance_charge_vals', [])" t-as="foreach_vals">
                    <cac:AllowanceCharge>
                        <t t-call="{{AllowanceChargeType_template}}">
                            <t t-set="vals" t-value="foreach_vals"/>
                        </t>
                    </cac:AllowanceCharge>
                </t>
            </t>
            <t t-foreach="vals.get('tax_total_vals', [])" t-as="foreach_vals">
                <cac:TaxTotal>
                    <t t-call="{{TaxTotalType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:TaxTotal>
            </t>
            <cac:Item>
                <t t-set="item_vals" t-value="vals.get('item_vals', {})"/>
                <cbc:Description t-esc="item_vals.get('description')"/>
                <cbc:Name t-esc="item_vals.get('name')"/>
                <cac:SellersItemIdentification>
                    <cbc:ID t-esc="item_vals.get('sellers_item_identification_vals', {}).get('id')"/>
                </cac:SellersItemIdentification>
                <t t-foreach="item_vals.get('classified_tax_category_vals', [])" t-as="foreach_vals">
                    <cac:ClassifiedTaxCategory>
                        <t t-call="{{TaxCategoryType_template}}">
                            <t t-set="vals" t-value="foreach_vals"/>
                        </t>
                    </cac:ClassifiedTaxCategory>
                </t>
            </cac:Item>
            <cac:Price>
                <t t-set="vals" t-value="vals.get('price_vals', {})"/>
                <cbc:PriceAmount
                        t-att-currencyID="vals['currency'].name"
                        t-esc="format_float(vals.get('price_amount'), vals.get('product_price_dp'))"/>
                <!-- nbr of item units to which the price applies), i.e.: 1 Dozen = 12 units, not mandatory -->
                <cbc:BaseQuantity
                        t-att="vals.get('base_quantity_attrs', {})"
                        t-esc="vals.get('base_quantity')"/>
            </cac:Price>
        </t>
    </template>

    <template id="ubl_20_InvoiceType">
        <t xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
           xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
           xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:UBLVersionID t-esc="vals.get('ubl_version_id')"/>
            <cbc:CustomizationID t-esc="vals.get('customization_id')"/>
            <cbc:ProfileID t-esc="vals.get('profile_id')"/>
            <cbc:ID t-esc="vals.get('id')"/>
            <cbc:IssueDate t-esc="vals.get('issue_date')"/>
            <cbc:InvoiceTypeCode
                    t-if="invoice.move_type == 'out_invoice'"
                    t-att="vals.get('invoice_type_code_attrs', {})"
                    t-esc="vals.get('invoice_type_code')"
            />
            <t t-foreach="vals.get('note_vals', [])" t-as="note">
                <cbc:Note t-esc="note"/>
            </t>
            <cbc:DocumentCurrencyCode
                    t-att="vals.get('document_currency_code_attrs', {})"
                    t-esc="invoice.currency_id.name.upper()"/>
            <t t-foreach="vals.get('invoice_period_vals_list', [])" t-as="foreach_vals">
                <cac:InvoicePeriod>
                    <cbc:StartDate t-esc="foreach_vals.get('start_date')"/>
                    <cbc:EndDate t-esc="foreach_vals.get('end_date')"/>
                </cac:InvoicePeriod>
            </t>
            <cac:OrderReference>
                <cbc:ID t-esc="vals.get('order_reference')"/>
                <cbc:SalesOrderID t-esc="vals.get('sales_order_id')"/>
            </cac:OrderReference>
            <cac:BillingReference t-if="vals.get('billing_reference_vals')">
                <cac:InvoiceDocumentReference>
                    <cbc:ID t-esc="vals['billing_reference_vals'].get('id')"/>
                    <cbc:IssueDate t-esc="vals['billing_reference_vals'].get('issue_date')"/>
                </cac:InvoiceDocumentReference>
            </cac:BillingReference>
            <cac:AccountingSupplierParty>
                <t t-set="accounting_supplier_vals" t-value="vals.get('accounting_supplier_party_vals', {})"/>
                <cac:Party>
                    <t t-call="{{PartyType_template}}">
                        <t t-set="vals" t-value="accounting_supplier_vals.get('party_vals', {})"/>
                    </t>
                </cac:Party>
            </cac:AccountingSupplierParty>
            <cac:AccountingCustomerParty>
                <t t-set="accounting_customer_vals" t-value="vals.get('accounting_customer_party_vals', {})"/>
                <cac:Party>
                    <t t-call="{{PartyType_template}}">
                        <t t-set="vals" t-value="accounting_customer_vals.get('party_vals', {})"/>
                    </t>
                </cac:Party>
            </cac:AccountingCustomerParty>
            <t id="delivery_foreach" t-foreach="vals.get('delivery_vals_list', [])" t-as="foreach_vals">
                <cac:Delivery>
                    <cbc:ActualDeliveryDate t-esc="foreach_vals.get('actual_delivery_date')"/>
                    <cac:DeliveryLocation>
                        <cac:Address>
                            <t t-call="{{AddressType_template}}">
                                <t t-set="vals"
                                   t-value="foreach_vals.get('delivery_location_vals', {}).get('delivery_address_vals', {})"/>
                            </t>
                        </cac:Address>
                    </cac:DeliveryLocation>
                </cac:Delivery>
            </t>
            <!-- In UBL 2.0 PaymentMeans is only present for Invoice
            while in UBL 2.1, it is present for Invoice and CreditNote
            http://www.datypic.com/sc/ubl20/e-ns19_Invoice.html
            http://www.datypic.com/sc/ubl20/e-ns14_CreditNote.html -->
            <t t-if="invoice.move_type == 'out_invoice'">
                <t t-foreach="vals.get('payment_means_vals_list', [])" t-as="foreach_vals">
                    <cac:PaymentMeans>
                        <t t-call="{{PaymentMeansType_template}}">
                            <t t-set="vals" t-value="foreach_vals"/>
                        </t>
                    </cac:PaymentMeans>
                </t>
            </t>
            <t t-foreach="vals.get('payment_terms_vals', [])" t-as="foreach_vals">
                <cac:PaymentTerms>
                    <t t-foreach="foreach_vals.get('note_vals', [])" t-as="note">
                        <cbc:Note t-esc="note"/>
                    </t>
                </cac:PaymentTerms>
            </t>
            <t t-foreach="vals.get('allowance_charge_vals', [])" t-as="foreach_vals">
                <cac:AllowanceCharge>
                    <t t-call="{{AllowanceChargeType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:AllowanceCharge>
            </t>
            <t t-foreach="vals.get('tax_total_vals', [])" t-as="foreach_vals">
                <cac:TaxTotal>
                    <t t-call="{{TaxTotalType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:TaxTotal>
            </t>
            <cac:LegalMonetaryTotal>
                <t t-set="monetary_tot_vals" t-value="vals.get('legal_monetary_total_vals', {})"/>
                <cbc:LineExtensionAmount
                    t-att-currencyID="monetary_tot_vals['currency'].name"
                    t-esc="format_float(monetary_tot_vals.get('line_extension_amount'), monetary_tot_vals.get('currency_dp'))"/>
                <cbc:TaxExclusiveAmount
                    t-att-currencyID="monetary_tot_vals['currency'].name"
                    t-esc="format_float(monetary_tot_vals.get('tax_exclusive_amount'), monetary_tot_vals.get('currency_dp'))"/>
                <cbc:TaxInclusiveAmount
                    t-att-currencyID="monetary_tot_vals['currency'].name"
                    t-esc="format_float(monetary_tot_vals.get('tax_inclusive_amount'), monetary_tot_vals.get('currency_dp'))"/>
                <cbc:AllowanceTotalAmount
                    t-att-currencyID="monetary_tot_vals['currency'].name"
                    t-esc="format_float(monetary_tot_vals.get('allowance_total_amount'), monetary_tot_vals.get('currency_dp'))"/>
                <cbc:PrepaidAmount
                    t-att-currencyID="monetary_tot_vals['currency'].name"
                    t-esc="format_float(monetary_tot_vals.get('prepaid_amount'), monetary_tot_vals.get('currency_dp'))"/>
                <cbc:PayableAmount
                    t-att-currencyID="monetary_tot_vals['currency'].name"
                    t-esc="format_float(monetary_tot_vals.get('payable_amount'), monetary_tot_vals.get('currency_dp'))"/>
            </cac:LegalMonetaryTotal>
            <t t-foreach="vals.get('invoice_line_vals', [])" t-as="foreach_vals">
                <cac:InvoiceLine t-if="invoice.move_type == 'out_invoice'">
                    <t t-call="{{InvoiceLineType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:InvoiceLine>
                <cac:CreditNoteLine t-if="invoice.move_type == 'out_refund'">
                    <t t-call="{{InvoiceLineType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:CreditNoteLine>
            </t>
        </t>
    </template>

    <template id="ubl_20_Invoice">
        <Invoice xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                 xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                 xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <t t-call="{{InvoiceType_template}}"/>
        </Invoice>
    </template>

    <template id="ubl_20_CreditNote">
        <CreditNote xmlns="urn:oasis:names:specification:ubl:schema:xsd:CreditNote-2"
                    xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                    xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <t t-call="{{InvoiceType_template}}"/>
        </CreditNote>
    </template>

</odoo>

```

## File: data\ubl_21_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="ubl_21_InvoiceLineType" inherit_id="account_edi_ubl_cii.ubl_20_InvoiceLineType" primary="True">
        <xpath expr="//*[local-name()='Item']" position="before">
            <t t-if="invoice.move_type == 'out_refund'">
                <!-- AllowanceCharge and TaxTotal are swapped in the CreditLine
                w.r.t InvoiceLine. see: http://www.datypic.com/sc/ubl21/e-cac_InvoiceLine.html
                vs http://www.datypic.com/sc/ubl21/e-cac_CreditNoteLine.html-->
                <t t-foreach="vals.get('allowance_charge_vals', [])" t-as="foreach_vals">
                    <cac:AllowanceCharge
                        xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                        xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                        xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                        <t t-call="{{AllowanceChargeType_template}}">
                            <t t-set="vals" t-value="foreach_vals"/>
                        </t>
                    </cac:AllowanceCharge>
                </t>
            </t>
        </xpath>
    </template>

    <template id="ubl_21_InvoiceType" inherit_id="account_edi_ubl_cii.ubl_20_InvoiceType" primary="True">
        <xpath expr="//*[local-name()='IssueDate']" position="after">
            <cbc:DueDate
                    xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                    xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                    xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                    t-if="invoice.move_type == 'out_invoice'"
                    t-esc="vals.get('due_date')"/>
            <cbc:CreditNoteTypeCode
                    xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                    xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                    xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                    t-if="invoice.move_type == 'out_refund'"
                    t-att="vals.get('invoice_type_code_attrs', {})"
                    t-esc="vals.get('credit_note_type_code')"
            />
        </xpath>
        <xpath expr="//*[local-name()='DocumentCurrencyCode']" position="after">
            <cbc:BuyerReference
                    xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                    xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                    xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                    t-esc="vals.get('buyer_reference')"/>
        </xpath>
        <xpath expr="//*[@id='delivery_foreach']" position="after">
            <t t-if="invoice.move_type == 'out_refund'">
                <t t-foreach="vals.get('payment_means_vals_list', [])" t-as="foreach_vals">
                    <cac:PaymentMeans
                        xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                        xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                        xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                        <t t-call="{{PaymentMeansType_template}}">
                            <t t-set="vals" t-value="foreach_vals"/>
                        </t>
                    </cac:PaymentMeans>
                </t>
            </t>
        </xpath>
    </template>

</odoo>

```

## File: models\account_edi_common.py

```python
# -*- coding: utf-8 -*-

from odoo import _, models
from odoo.tools import float_repr
from odoo.tests.common import Form
from odoo.exceptions import UserError, ValidationError
from odoo.tools.float_utils import float_round
from odoo.tools.misc import formatLang

from zeep import Client

# -------------------------------------------------------------------------
# UNIT OF MEASURE
# -------------------------------------------------------------------------
UOM_TO_UNECE_CODE = {
    'uom.product_uom_unit': 'C62',
    'uom.product_uom_dozen': 'DZN',
    'uom.product_uom_kgm': 'KGM',
    'uom.product_uom_gram': 'GRM',
    'uom.product_uom_day': 'DAY',
    'uom.product_uom_hour': 'HUR',
    'uom.product_uom_ton': 'TNE',
    'uom.product_uom_meter': 'MTR',
    'uom.product_uom_km': 'KMT',
    'uom.product_uom_cm': 'CMT',
    'uom.product_uom_litre': 'LTR',
    'uom.product_uom_cubic_meter': 'MTQ',
    'uom.product_uom_lb': 'LBR',
    'uom.product_uom_oz': 'ONZ',
    'uom.product_uom_inch': 'INH',
    'uom.product_uom_foot': 'FOT',
    'uom.product_uom_mile': 'SMI',
    'uom.product_uom_floz': 'OZA',
    'uom.product_uom_qt': 'QT',
    'uom.product_uom_gal': 'GLL',
    'uom.product_uom_cubic_inch': 'INQ',
    'uom.product_uom_cubic_foot': 'FTQ',
}

# -------------------------------------------------------------------------
# ELECTRONIC ADDRESS SCHEME (EAS), see https://docs.peppol.eu/poacc/billing/3.0/codelist/eas/
# -------------------------------------------------------------------------
COUNTRY_EAS = {
    'HU': 9910,
    'AT': 9915,
    'ES': 9920,
    'AD': 9922,
    'AL': 9923,
    'BA': 9924,
    'BE': 9925,
    'BG': 9926,
    'CH': 9927,
    'CY': 9928,
    'CZ': 9929,
    'DE': 9930,
    'DK': '0184',
    'EE': 9931,
    'GB': 9932,
    'GR': 9933,
    'HR': 9934,
    'IE': 9935,
    'IT': '0211',
    'LI': 9936,
    'LT': 9937,
    'LU': 9938,
    'LV': 9939,
    'MC': 9940,
    'ME': 9941,
    'MK': 9942,
    'MT': 9943,
    'NL': 9944,
    'PL': 9945,
    'PT': 9946,
    'RO': 9947,
    'RS': 9948,
    'SI': 9949,
    'SK': 9950,
    'SM': 9951,
    'TR': 9952,
    'VA': 9953,
    'SE': 9955,
    'FR': 9957,
    'NO': '0192',
    'SG': '0195',
    'AU': '0151',
    'FI': '0213',
}


class AccountEdiCommon(models.AbstractModel):
    _name = "account.edi.common"
    _description = "Common functions for EDI documents: generate the data, the constraints, etc"

    # -------------------------------------------------------------------------
    # HELPERS
    # -------------------------------------------------------------------------

    def format_float(self, amount, precision_digits):
        if amount is None:
            return None
        return float_repr(float_round(amount, precision_digits), precision_digits)

    def _get_uom_unece_code(self, line):
        """
        list of codes: https://docs.peppol.eu/poacc/billing/3.0/codelist/UNECERec20/
        or https://unece.org/fileadmin/DAM/cefact/recommendations/bkup_htm/add2c.htm (sorted by letter)
        """
        xmlid = line.product_uom_id.get_external_id()
        if xmlid and line.product_uom_id.id in xmlid:
            return UOM_TO_UNECE_CODE.get(xmlid[line.product_uom_id.id], 'C62')
        return 'C62'

    def _find_value(self, xpath, tree):
        # avoid 'TypeError: empty namespace prefix is not supported in XPath'
        nsmap = {k: v for k, v in tree.nsmap.items() if k is not None}
        return self.env['account.edi.format']._find_value(xpath=xpath, xml_element=tree, namespaces=nsmap)

    # -------------------------------------------------------------------------
    # TAXES
    # -------------------------------------------------------------------------

    def _validate_taxes(self, invoice):
        """ Validate the structure of the tax repartition lines (invalid structure could lead to unexpected results)
        """
        for tax in invoice.invoice_line_ids.tax_ids:
            try:
                tax._validate_repartition_lines()
            except ValidationError as e:
                error_msg = _("Tax '%s' is invalid: %s", tax.name, e.args[0])  # args[0] gives the error message
                raise ValidationError(error_msg)

    def _get_tax_unece_codes(self, invoice, tax):
        """
        Source: doc of Peppol (but the CEF norm is also used by factur-x, yet not detailed)
        https://docs.peppol.eu/poacc/billing/3.0/syntax/ubl-invoice/cac-TaxTotal/cac-TaxSubtotal/cac-TaxCategory/cbc-TaxExemptionReasonCode/
        https://docs.peppol.eu/poacc/billing/3.0/codelist/vatex/
        https://docs.peppol.eu/poacc/billing/3.0/codelist/UNCL5305/
        :returns: {
            tax_category_code: str,
            tax_exemption_reason_code: str,
            tax_exemption_reason: str,
        }
        """
        def create_dict(tax_category_code=None, tax_exemption_reason_code=None, tax_exemption_reason=None):
            return {
                'tax_category_code': tax_category_code,
                'tax_exemption_reason_code': tax_exemption_reason_code,
                'tax_exemption_reason': tax_exemption_reason,
            }

        supplier = invoice.company_id.partner_id.commercial_partner_id
        customer = invoice.commercial_partner_id

        # add Norway, Iceland, Liechtenstein
        european_economic_area = self.env.ref('base.europe').country_ids.mapped('code') + ['NO', 'IS', 'LI']

        if customer.country_id.code == 'ES' and customer.zip:
            if customer.zip[:2] in ('35', '38'):  # Canary
                # [BR-IG-10]-A VAT breakdown (BG-23) with VAT Category code (BT-118) "IGIC" shall not have a VAT
                # exemption reason code (BT-121) or VAT exemption reason text (BT-120).
                return create_dict(tax_category_code='L')
            if customer.zip[:2] in ('51', '52'):
                return create_dict(tax_category_code='M')  # Ceuta & Mellila

        # see: https://anskaffelser.dev/postaward/g3/spec/current/billing-3.0/norway/#_value_added_tax_norwegian_mva
        if customer.country_id.code == 'NO':
            if tax.amount == 25:
                return create_dict(tax_category_code='S', tax_exemption_reason=_('Output VAT, regular rate'))
            if tax.amount == 15:
                return create_dict(tax_category_code='S', tax_exemption_reason=_('Output VAT, reduced rate, middle'))
            if tax.amount == 11.11:
                return create_dict(tax_category_code='S', tax_exemption_reason=_('Output VAT, reduced rate, raw fish'))
            if tax.amount == 12:
                return create_dict(tax_category_code='S', tax_exemption_reason=_('Output VAT, reduced rate, low'))

        if supplier.country_id == customer.country_id:
            if not tax or tax.amount == 0:
                # in theory, you should indicate the precise law article
                return create_dict(tax_category_code='E', tax_exemption_reason=_('Articles 226 items 11 to 15 Directive 2006/112/EN'))
            else:
                return create_dict(tax_category_code='S')  # standard VAT

        if supplier.country_id.code in european_economic_area:
            if tax.amount != 0:
                # otherwise, the validator will complain because G and K code should be used with 0% tax
                return create_dict(tax_category_code='S')
            if customer.country_id.code not in european_economic_area:
                return create_dict(
                    tax_category_code='G',
                    tax_exemption_reason_code='VATEX-EU-G',
                    tax_exemption_reason=_('Export outside the EU'),
                )
            if customer.country_id.code in european_economic_area:
                return create_dict(
                    tax_category_code='K',
                    tax_exemption_reason_code='VATEX-EU-IC',
                    tax_exemption_reason=_('Intra-Community supply'),
                )

        if tax.amount != 0:
            return create_dict(tax_category_code='S')
        else:
            return create_dict(tax_category_code='E', tax_exemption_reason=_('Articles 226 items 11 to 15 Directive 2006/112/EN'))

    def _get_tax_category_list(self, invoice, taxes):
        """ Full list: https://unece.org/fileadmin/DAM/trade/untdid/d16b/tred/tred5305.htm
        Subset: https://docs.peppol.eu/poacc/billing/3.0/codelist/UNCL5305/

        :param taxes:   account.tax records.
        :return:        A list of values to fill the TaxCategory foreach template.
        """
        res = []
        for tax in taxes:
            tax_unece_codes = self._get_tax_unece_codes(invoice, tax)
            res.append({
                'id': tax_unece_codes.get('tax_category_code'),
                'percent': tax.amount if tax.amount_type == 'percent' else False,
                'name': tax_unece_codes.get('tax_exemption_reason'),
                **tax_unece_codes,
            })
        return res

    # -------------------------------------------------------------------------
    # CONSTRAINTS
    # -------------------------------------------------------------------------

    def _check_required_fields(self, record, field_names, custom_warning_message=""):
        """
        This function check that a field exists on a record or dictionaries
        returns a generic error message if it's not the case or a custom one if specified
        """
        if not record:
            return custom_warning_message or _("The element %s is required on %s.", record, ', '.join(field_names))

        if not isinstance(field_names, list):
            field_names = [field_names]

        has_values = any(record[field_name] for field_name in field_names)
        # field is present
        if has_values:
            return

        # field is not present
        if custom_warning_message or isinstance(record, dict):
            return custom_warning_message or _("The element %s is required on %s.", record, ', '.join(field_names))

        display_field_names = record.fields_get(field_names)
        if len(field_names) == 1:
            display_field = f"'{display_field_names[field_names[0]]['string']}'"
            return _("The field %s is required on %s.", display_field, record.display_name)
        else:
            display_fields = ', '.join(f"'{display_field_names[x]['string']}'" for x in display_field_names)
            return _("At least one of the following fields %s is required on %s.", display_fields, record.display_name)

    # -------------------------------------------------------------------------
    # COMMON CONSTRAINTS
    # -------------------------------------------------------------------------

    def _invoice_constraints_common(self, invoice):
        # check that there is a tax on each line
        for line in invoice.invoice_line_ids.filtered(lambda x: not x.display_type):
            if not line.tax_ids:
                return {'tax_on_line': _("Each invoice line should have at least one tax.")}
        return {}

    # -------------------------------------------------------------------------
    # Import invoice
    # -------------------------------------------------------------------------

    def _import_invoice(self, journal, filename, tree, existing_invoice=None):
        move_types, qty_factor = self._get_import_document_amount_sign(filename, tree)
        if not move_types:
            return
        if journal.type == 'purchase':
            move_type = move_types[0]
        elif journal.type == 'sale':
            move_type = move_types[1]
        else:
            return
        if existing_invoice and existing_invoice.move_type != move_type:
            # with an email alias to create account_move, first the move is created (using alias_defaults, which
            # contains move_type = 'out_invoice') then the attachment is decoded, if it represents a credit note,
            # the move type needs to be changed to 'out_refund'
            types = {move_type, existing_invoice.move_type}
            if types == {'out_invoice', 'out_refund'} or types == {'in_invoice', 'in_refund'}:
                existing_invoice.move_type = move_type
            else:
                return

        invoice = existing_invoice or self.env['account.move']
        invoice_form = Form(invoice.with_context(
            account_predictive_bills_disable_prediction=True,
            default_move_type=move_type,
            default_journal_id=journal.id,
        ))
        invoice_form, logs = self._import_fill_invoice_form(journal, tree, invoice_form, qty_factor)
        invoice = invoice_form.save()
        if invoice:
            if logs:
                body = _(
                    "<strong>Format used to import the invoice: %s</strong> <p><li> %s </li></p>",
                    str(self._description), "</li><li>".join(logs)
                )
            else:
                body = _("<strong>Format used to import the invoice: %s</strong>", str(self._description))
            invoice.with_context(no_new_invoice=True).message_post(body=body)

        # === Import the embedded PDF in the xml if some are found ===

        attachments = self.env['ir.attachment']
        additional_docs = tree.findall('./{*}AdditionalDocumentReference')
        for document in additional_docs:
            attachment_name = document.find('{*}ID')
            attachment_data = document.find('{*}Attachment/{*}EmbeddedDocumentBinaryObject')
            if attachment_name is not None \
                    and attachment_data is not None \
                    and attachment_data.attrib.get('mimeCode') == 'application/pdf':
                text = attachment_data.text
                # Normalize the name of the file : some e-fff emitters put the full path of the file
                # (Windows or Linux style) and/or the name of the xml instead of the pdf.
                # Get only the filename with a pdf extension.
                name = (attachment_name.text or 'invoice').split('\\')[-1].split('/')[-1].split('.')[0] + '.pdf'
                attachment = self.env['ir.attachment'].create({
                    'name': name,
                    'res_id': invoice.id,
                    'res_model': 'account.move',
                    'datas': text + '=' * (len(text) % 3),  # Fix incorrect padding
                    'type': 'binary',
                    'mimetype': 'application/pdf',
                })
                # Upon receiving an email (containing an xml) with a configured alias to create invoice, the xml is
                # set as the main_attachment. To be rendered in the form view, the pdf should be the main_attachment.
                if invoice.message_main_attachment_id and \
                        invoice.message_main_attachment_id.name.endswith('.xml') and \
                        'pdf' not in invoice.message_main_attachment_id.mimetype:
                    invoice.message_main_attachment_id = attachment
                attachments |= attachment
        if attachments:
            invoice.with_context(no_new_invoice=True).message_post(attachment_ids=attachments.ids)

        return invoice

    def _import_retrieve_and_fill_partner(self, invoice, name, phone, mail, vat):
        """ Retrieve the partner, if no matching partner is found, create it (only if he has a vat and a name)
        """
        invoice.partner_id = self.env['account.edi.format']._retrieve_partner(name=name, phone=phone, mail=mail, vat=vat)
        if not invoice.partner_id and name and vat:
            invoice.partner_id = self.env['res.partner'].create({'name': name, 'email': mail, 'phone': phone})
            # an invalid VAT will throw a ValidationError (see 'check_vat' in base_vat)
            try:
                invoice.partner_id.vat = vat
            except ValidationError:
                invoice.partner_id.vat = False

    def _import_fill_invoice_allowance_charge(self, tree, invoice_form, journal, qty_factor):
        logs = []
        if '{urn:oasis:names:specification:ubl:schema:xsd' in tree.tag:
            is_ubl = True
        elif '{urn:un:unece:uncefact:data:standard:' in tree.tag:
            is_ubl = False
        else:
            return

        xpath = './{*}AllowanceCharge' if is_ubl else './{*}SupplyChainTradeTransaction/{*}ApplicableHeaderTradeSettlement/{*}SpecifiedTradeAllowanceCharge'
        allowance_charge_nodes = tree.findall(xpath)
        for allow_el in allowance_charge_nodes:
            with invoice_form.invoice_line_ids.new() as invoice_line_form:
                invoice_line_form.sequence = 0  # be sure to put these lines above the 'real' invoice lines

                charge_factor = -1  # factor is -1 for discount, 1 for charge
                if is_ubl:
                    charge_indicator_node = allow_el.find('./{*}ChargeIndicator')
                else:
                    charge_indicator_node = allow_el.find('./{*}ChargeIndicator/{*}Indicator')
                if charge_indicator_node is not None:
                    charge_factor = -1 if charge_indicator_node.text == 'false' else 1

                name = ""
                reason_code_node = allow_el.find('./{*}AllowanceChargeReasonCode' if is_ubl else './{*}ReasonCode')
                if reason_code_node is not None:
                    name += reason_code_node.text + " "
                reason_node = allow_el.find('./{*}AllowanceChargeReason' if is_ubl else './{*}Reason')
                if reason_node is not None:
                    name += reason_node.text
                invoice_line_form.name = name

                amount_node = allow_el.find('./{*}Amount' if is_ubl else './{*}ActualAmount')
                base_amount_node = allow_el.find('./{*}BaseAmount' if is_ubl else './{*}BasisAmount')
                # Since there is no quantity associated for the allowance/charge on document level,
                # if we have an invoice with negative amounts, the price was multiplied by -1 and not the quantity
                # See the file in test_files: 'base-negative-inv-correction.xml' VS 'base-example.xml' for 'Insurance'
                if base_amount_node is not None:
                    invoice_line_form.price_unit = float(base_amount_node.text) * charge_factor * qty_factor
                    percent_node = allow_el.find('./{*}MultiplierFactorNumeric' if is_ubl else './{*}CalculationPercent')
                    if percent_node is not None:
                        invoice_line_form.quantity = float(percent_node.text) / 100
                elif amount_node is not None:
                    invoice_line_form.price_unit = float(amount_node.text) * charge_factor * qty_factor

                invoice_line_form.tax_ids.clear()  # clear the default taxes applied to the line
                tax_xpath = './{*}TaxCategory/{*}Percent' if is_ubl else './{*}CategoryTradeTax/{*}RateApplicablePercent'
                for tax_categ_percent_el in allow_el.findall(tax_xpath):
                    tax = self.env['account.tax'].search([
                        ('company_id', '=', journal.company_id.id),
                        ('amount', '=', float(tax_categ_percent_el.text)),
                        ('amount_type', '=', 'percent'),
                        ('type_tax_use', '=', journal.type),
                    ], limit=1)
                    if tax:
                        invoice_line_form.tax_ids.add(tax)
                    else:
                        logs.append(
                            _("Could not retrieve the tax: %s %% for line '%s'.",
                              float(tax_categ_percent_el.text),
                              name)
                        )
        return logs

    def _import_fill_invoice_down_payment(self, invoice_form, prepaid_node, qty_factor):
        """
        DEPRECATED: removed in master
        Creates a down payment line on the invoice at import if prepaid_node (TotalPrepaidAmount in CII,
        PrepaidAmount in UBL) exists.
        qty_factor -1 if the xml is labelled as an invoice but has negative amounts -> conversion into a credit note
        needed, so we need this multiplier. Otherwise, qty_factor is 1.
        """
        if prepaid_node is not None and float(prepaid_node.text) != 0:
            # create a section
            with invoice_form.invoice_line_ids.new() as invoice_line_form:
                invoice_line_form.sequence = 998
                invoice_line_form.display_type = 'line_section'
                invoice_line_form.name = _("Down Payments")
                invoice_line_form.price_unit = 0
                invoice_line_form.quantity = 0
                invoice_line_form.account_id = self.env['account.account']
            # create the line with the down payment
            with invoice_form.invoice_line_ids.new() as invoice_line_form:
                invoice_line_form.sequence = 999
                invoice_line_form.name = _("Down Payment")
                invoice_line_form.price_unit = float(prepaid_node.text)
                invoice_line_form.quantity = qty_factor * -1
                invoice_line_form.tax_ids.clear()

    def _import_log_prepaid_amount(self, invoice_form, prepaid_node, qty_factor):
        """
        Log a message in the chatter at import if prepaid_node (TotalPrepaidAmount in CII, PrepaidAmount in UBL) exists.
        """
        prepaid_amount = float(prepaid_node.text) if prepaid_node is not None else 0.0
        if not invoice_form.currency_id.is_zero(prepaid_amount):
            amount = prepaid_amount * qty_factor
            formatted_amount = formatLang(self.env, amount, currency_obj=invoice_form.currency_id)
            return [
                _("A payment of %s was detected.", formatted_amount)
            ]
        return []

    def _import_fill_invoice_line_values(self, tree, xpath_dict, invoice_line_form, qty_factor):
        """
        Read the xml invoice, extract the invoice line values, compute the odoo values
        to fill an invoice line form: quantity, price_unit, discount, product_uom_id.

        The way of computing invoice line is quite complicated:
        https://docs.peppol.eu/poacc/billing/3.0/bis/#_calculation_on_line_level (same as in factur-x documentation)

        line_net_subtotal = ( gross_unit_price - rebate ) * (billed_qty / basis_qty) - allow_charge_amount

        with (UBL | CII):
            * net_unit_price = 'Price/PriceAmount' | 'NetPriceProductTradePrice' (mandatory) (BT-146)
            * gross_unit_price = 'Price/AllowanceCharge/BaseAmount' | 'GrossPriceProductTradePrice' (optional) (BT-148)
            * basis_qty = 'Price/BaseQuantity' | 'BasisQuantity' (optional, either below net_price node or
                gross_price node) (BT-149)
            * billed_qty = 'InvoicedQuantity' | 'BilledQuantity' (mandatory) (BT-129)
            * allow_charge_amount = sum of 'AllowanceCharge' | 'SpecifiedTradeAllowanceCharge' (same level as Price)
                ON THE LINE level (optional) (BT-136 / BT-141)
            * line_net_subtotal = 'LineExtensionAmount' | 'LineTotalAmount' (mandatory) (BT-131)
            * rebate = 'Price/AllowanceCharge' | 'AppliedTradeAllowanceCharge' below gross_price node ! (BT-147)
                "item price discount" which is different from the usual allow_charge_amount
                gross_unit_price (BT-148) - rebate (BT-147) = net_unit_price (BT-146)

        In Odoo, we obtain:
        (1) = price_unit  =  gross_price_unit / basis_qty  =  (net_price_unit + rebate) / basis_qty
        (2) = quantity  =  billed_qty
        (3) = discount (converted into a percentage)  =  100 * (1 - price_subtotal / (billed_qty * price_unit))
        (4) = price_subtotal

        Alternatively, we could also set: quantity = billed_qty/basis_qty

        WARNING, the basis quantity parameter is annoying, for instance, an invoice with a line:
            item A  | price per unit of measure/unit price: 30  | uom = 3 pieces | billed qty = 3 | rebate = 2  | untaxed total = 28
        Indeed, 30 $ / 3 pieces = 10 $ / piece => 10 * 3 (billed quantity) - 2 (rebate) = 28

        UBL ROUNDING: "the result of Item line net
            amount = ((Item net price (BT-146)÷Item price base quantity (BT-149))×(Invoiced Quantity (BT-129))
        must be rounded to two decimals, and the allowance/charge amounts are also rounded separately."
        It is not possible to do it in Odoo.

        :params tree
        :params xpath_dict dict: {
            'basis_qty': list of str,
            'gross_price_unit': str,
            'rebate': str,
            'net_price_unit': str,
            'billed_qty': str,
            'allowance_charge': str, to be used in a findall !,
            'allowance_charge_indicator': str, relative xpath from allowance_charge,
            'allowance_charge_amount': str, relative xpath from allowance_charge,
            'line_total_amount': str,
        }
        :params: invoice_line_form
        :params: qty_factor
        :returns: {
            'quantity': float,
            'product_uom_id': (optional) uom.uom,
            'price_unit': float,
            'discount': float,
        }
        """
        # basis_qty (optional)
        basis_qty = 1
        for xpath in xpath_dict['basis_qty']:
            basis_quantity_node = tree.find(xpath)
            if basis_quantity_node is not None:
                basis_qty = float(basis_quantity_node.text) or 1

        # gross_price_unit (optional)
        gross_price_unit = None
        gross_price_unit_node = tree.find(xpath_dict['gross_price_unit'])
        if gross_price_unit_node is not None:
            gross_price_unit = float(gross_price_unit_node.text)

        # rebate (optional)
        # Discount. /!\ as no percent discount can be set on a line, need to infer the percentage
        # from the amount of the actual amount of the discount (the allowance charge)
        rebate = 0
        rebate_node = tree.find(xpath_dict['rebate'])
        net_price_unit_node = tree.find(xpath_dict['net_price_unit'])
        if rebate_node is not None:
            rebate = float(rebate_node.text)
        elif net_price_unit_node is not None and gross_price_unit_node is not None:
            rebate = float(gross_price_unit_node.text) - float(net_price_unit_node.text)

        # net_price_unit (mandatory)
        net_price_unit = None
        if net_price_unit_node is not None:
            net_price_unit = float(net_price_unit_node.text)

        # billed_qty (mandatory)
        billed_qty = 1
        product_uom_id = None
        quantity_node = tree.find(xpath_dict['billed_qty'])
        if quantity_node is not None:
            billed_qty = float(quantity_node.text)
            uom_xml = quantity_node.attrib.get('unitCode')
            if uom_xml:
                uom_infered_xmlid = [
                    odoo_xmlid for odoo_xmlid, uom_unece in UOM_TO_UNECE_CODE.items() if uom_unece == uom_xml
                ]
                if uom_infered_xmlid:
                    product_uom_id = self.env.ref(uom_infered_xmlid[0], raise_if_not_found=False)

        # allow_charge_amount
        fixed_taxes_list = []
        allow_charge_amount = 0  # if positive: it's a discount, if negative: it's a charge
        allow_charge_nodes = tree.findall(xpath_dict['allowance_charge'])
        for allow_charge_el in allow_charge_nodes:
            charge_indicator = allow_charge_el.find(xpath_dict['allowance_charge_indicator'])
            if charge_indicator.text and charge_indicator.text.lower() == 'false':
                discount_factor = 1  # it's a discount
            else:
                discount_factor = -1  # it's a charge
            amount = allow_charge_el.find(xpath_dict['allowance_charge_amount'])
            reason_code = allow_charge_el.find(xpath_dict['allowance_charge_reason_code'])
            reason = allow_charge_el.find(xpath_dict['allowance_charge_reason'])
            if amount is not None:
                if reason_code is not None and reason_code.text == 'AEO' and reason is not None:
                    # Handle Fixed Taxes: when exporting from Odoo, we use the allowance_charge node
                    fixed_taxes_list.append({
                        'tax_name': reason.text,
                        'tax_amount': float(amount.text),
                    })
                else:
                    allow_charge_amount += float(amount.text) * discount_factor

        # line_net_subtotal (mandatory)
        price_subtotal = None
        line_total_amount_node = tree.find(xpath_dict['line_total_amount'])
        if line_total_amount_node is not None:
            price_subtotal = float(line_total_amount_node.text)

        ####################################################
        # Setting the values on the invoice_line_form
        ####################################################

        # quantity
        quantity = billed_qty * qty_factor

        # price_unit
        if gross_price_unit is not None:
            price_unit = gross_price_unit / basis_qty
        elif net_price_unit is not None:
            price_unit = (net_price_unit + rebate) / basis_qty
        elif price_subtotal is not None:
            price_unit = (price_subtotal + allow_charge_amount) / (billed_qty or 1)
        else:
            raise UserError(_("No gross price, net price nor line subtotal amount found for line in xml"))

        # discount
        discount = 0
        amount_fixed_taxes = sum(d['tax_amount'] for d in fixed_taxes_list)
        if billed_qty * price_unit != 0 and price_subtotal is not None:
            discount = 100 * (1 - (price_subtotal - amount_fixed_taxes) / (billed_qty * price_unit))

        # Sometimes, the xml received is very bad: unit price = 0, qty = 1, but price_subtotal = -200
        # for instance, when filling a down payment as an invoice line. The equation in the docstring is not
        # respected, and the result will not be correct, so we just follow the simple rule below:
        if net_price_unit == 0 and price_subtotal != net_price_unit * (billed_qty / basis_qty) - allow_charge_amount:
            price_unit = price_subtotal / (billed_qty or 1)

        return {
            'quantity': quantity,
            'price_unit': price_unit,
            'discount': discount,
            'product_uom_id': product_uom_id,
            'fixed_taxes_list': fixed_taxes_list,
        }

    def _import_retrieve_fixed_tax(self, invoice_line_form, fixed_tax_vals):
        """ Retrieve the fixed tax at import, iteratively search for a tax:
        1. not price_include matching the name and the amount
        2. not price_include matching the amount
        3. price_include matching the name and the amount
        4. price_include matching the amount
        """
        base_domain = [
            ('company_id', '=', invoice_line_form.company_id.id),
            ('amount_type', '=', 'fixed'),
            ('amount', '=', fixed_tax_vals['tax_amount']),
        ]
        for price_include in (False, True):
            for name in (fixed_tax_vals['tax_name'], False):
                domain = base_domain + [('price_include', '=', price_include)]
                if name:
                    domain.append(('name', '=', name))
                tax = self.env['account.tax'].search(domain, limit=1)
                if tax:
                    return tax
        return self.env['account.tax']

    def _import_fill_invoice_line_taxes(self, journal, tax_nodes, invoice_line_form, inv_line_vals, logs):
        # Taxes: all amounts are tax excluded, so first try to fetch price_include=False taxes,
        # if no results, try to fetch the price_include=True taxes. If results, need to adapt the price_unit.
        inv_line_vals['taxes'] = []
        for tax_node in tax_nodes:
            amount = float(tax_node.text)
            domain = [
                ('company_id', '=', journal.company_id.id),
                ('amount_type', '=', 'percent'),
                ('type_tax_use', '=', journal.type),
                ('amount', '=', amount),
            ]
            tax_excl = self.env['account.tax'].search(domain + [('price_include', '=', False)], limit=1)
            tax_incl = self.env['account.tax'].search(domain + [('price_include', '=', True)], limit=1)
            if tax_excl:
                inv_line_vals['taxes'].append(tax_excl)
            elif tax_incl:
                inv_line_vals['taxes'].append(tax_incl)
                inv_line_vals['price_unit'] *= (1 + tax_incl.amount / 100)
            else:
                logs.append(_("Could not retrieve the tax: %s %% for line '%s'.", amount, invoice_line_form.name))

        # Handle Fixed Taxes
        for fixed_tax_vals in inv_line_vals['fixed_taxes_list']:
            tax = self._import_retrieve_fixed_tax(invoice_line_form, fixed_tax_vals)
            if not tax:
                # Nothing found: fix the price_unit s.t. line subtotal is matching the original invoice
                inv_line_vals['price_unit'] += fixed_tax_vals['tax_amount']
            elif tax.price_include:
                inv_line_vals['taxes'].append(tax)
                inv_line_vals['price_unit'] += tax.amount
            else:
                inv_line_vals['taxes'].append(tax)

        # Set the values on the line_form
        invoice_line_form.quantity = inv_line_vals['quantity']
        if not inv_line_vals.get('product_uom_id'):
            logs.append(
                _("Could not retrieve the unit of measure for line with label '%s'.", invoice_line_form.name))
        elif not invoice_line_form.product_id:
            # no product set on the line, no need to check uom compatibility
            invoice_line_form.product_uom_id = inv_line_vals['product_uom_id']
        elif inv_line_vals['product_uom_id'].category_id == invoice_line_form.product_id.product_tmpl_id.uom_id.category_id:
            # needed to check that the uom is compatible with the category of the product
            invoice_line_form.product_uom_id = inv_line_vals['product_uom_id']

        invoice_line_form.price_unit = inv_line_vals['price_unit']
        invoice_line_form.discount = inv_line_vals['discount']
        invoice_line_form.tax_ids.clear()
        for tax in inv_line_vals['taxes']:
            invoice_line_form.tax_ids.add(tax)
        return logs

    # -------------------------------------------------------------------------
    # Check xml using the free API from Ph. Helger, don't abuse it !
    # -------------------------------------------------------------------------

    def _check_xml_ecosio(self, invoice, xml_content, ecosio_formats):
        # see https://peppol.helger.com/public/locale-en_US/menuitem-validation-ws2
        if not ecosio_formats:
            return
        soap_client = Client('https://peppol.helger.com/wsdvs?wsdl')
        if invoice.move_type == 'out_invoice':
            ecosio_format = ecosio_formats['invoice']
        elif invoice.move_type == 'out_refund':
            ecosio_format = ecosio_formats['credit_note']
        else:
            invoice.with_context(no_new_invoice=True).message_post(
                body="ECOSIO: could not validate xml, formats only exist for invoice or credit notes"
            )
            return
        if not ecosio_format:
            return
        response = soap_client.service.validate(xml_content, ecosio_format)

        report = []
        errors_cnt = 0
        for item in response['Result']:
            if item['artifactPath']:
                report.append(
                    "<li><font style='color:Blue;'><strong>" + item['artifactPath'] + "</strong></font></li>")
            for detail in item['Item']:
                if detail['errorLevel'] == 'WARN':
                    errors_cnt += 1
                    report.append(
                        "<li><font style='color:Orange;'><strong>" + detail['errorText'] + "</strong></font></li>")
                elif detail['errorLevel'] == 'ERROR':
                    errors_cnt += 1
                    report.append(
                        "<li><font style='color:Tomato;'><strong>" + detail['errorText'] + "</strong></font></li>")

        if errors_cnt == 0:
            invoice.with_context(no_new_invoice=True).message_post(
                body=f"<font style='color:Green;'><strong>ECOSIO: All clear for format {ecosio_format}!</strong></font>"
            )
        else:
            invoice.with_context(no_new_invoice=True).message_post(
                body=f"<font style='color:Tomato;'><strong>ECOSIO ERRORS/WARNINGS for format {ecosio_format}</strong></font>: <ul> "
                     + "\n".join(report) + " </ul>"
            )
        return response

```

## File: models\account_edi_format.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, _
from odoo.tools import str2bool
from odoo.addons.account_edi_ubl_cii.models.account_edi_common import COUNTRY_EAS

import logging

_logger = logging.getLogger(__name__)

FORMAT_CODES = [
    'facturx_1_0_05',
    'ubl_bis3',
    'ubl_de',
    'nlcius_1',
    'efff_1',
    'ubl_2_1',
    'ehf_3',
]


class AccountEdiFormat(models.Model):
    _inherit = 'account.edi.format'

    ####################################################
    # Helpers
    ####################################################

    def _infer_xml_builder_from_tree(self, tree):
        self.ensure_one()
        ubl_version = tree.find('{*}UBLVersionID')
        customization_id = tree.find('{*}CustomizationID')
        if tree.tag == '{urn:un:unece:uncefact:data:standard:CrossIndustryInvoice:100}CrossIndustryInvoice':
            return self.env['account.edi.xml.cii']
        if ubl_version is not None:
            if ubl_version.text == '2.0':
                return self.env['account.edi.xml.ubl_20']
            if ubl_version.text in ('2.1', '2.2', '2.3'):
                return self.env['account.edi.xml.ubl_21']
        if customization_id is not None:
            if 'xrechnung' in customization_id.text:
                return self.env['account.edi.xml.ubl_de']
            if customization_id.text == 'urn:cen.eu:en16931:2017#compliant#urn:fdc:nen.nl:nlcius:v1.0':
                return self.env['account.edi.xml.ubl_nl']
            # Allow to parse any format derived from the european semantic norm EN16931
            if 'urn:cen.eu:en16931:2017' in customization_id.text:
                return self.env['account.edi.xml.ubl_bis3']
        return

    def _get_xml_builder(self, company):
        # see https://communaute.chorus-pro.gouv.fr/wp-content/uploads/2017/08/20170630_Solution-portail_Dossier_Specifications_Fournisseurs_Chorus_Facture_V.1.pdf
        # page 45 -> ubl 2.1 for France seems also supported
        if self.code == 'facturx_1_0_05':
            return self.env['account.edi.xml.cii']
        # if the company's country is not in the EAS mapping, nothing is generated
        if self.code == 'ubl_bis3' and company.country_id.code in COUNTRY_EAS:
            return self.env['account.edi.xml.ubl_bis3']
        # the EDI option will only appear on the journal of german companies
        if self.code == 'ubl_de' and company.country_id.code == 'DE':
            return self.env['account.edi.xml.ubl_de']
        # the EDI option will only appear on the journal of belgian companies
        if self.code == 'efff_1' and company.country_id.code == 'BE':
            return self.env['account.edi.xml.ubl_efff']

    def _is_ubl_cii_available(self, company):
        """
        Returns a boolean indicating whether it is possible to generate an xml file using one of the formats from this
        module or not
        """
        return self._get_xml_builder(company) is not None

    ####################################################
    # Export: Account.edi.format override
    ####################################################

    def _is_required_for_invoice(self, invoice):
        # EXTENDS account_edi
        self.ensure_one()
        if self.code not in FORMAT_CODES:
            return super()._is_required_for_invoice(invoice)

        return self._is_ubl_cii_available(invoice.company_id) and invoice.move_type in ('out_invoice', 'out_refund')

    def _is_compatible_with_journal(self, journal):
        # EXTENDS account_edi
        # the formats appear on the journal only if they are compatible (e.g. NLCIUS only appear for dutch companies)
        self.ensure_one()
        if self.code not in FORMAT_CODES:
            return super()._is_compatible_with_journal(journal)
        return self._is_ubl_cii_available(journal.company_id) and journal.type == 'sale'

    def _is_enabled_by_default_on_journal(self, journal):
        # EXTENDS account_edi
        # only facturx is enabled by default, the other formats aren't
        self.ensure_one()
        if self.code not in FORMAT_CODES:
            return super()._is_enabled_by_default_on_journal(journal)
        return self.code == 'facturx_1_0_05'

    def _post_invoice_edi(self, invoices, test_mode=False):
        # EXTENDS account_edi
        self.ensure_one()

        if self.code not in FORMAT_CODES:
            return super()._post_invoice_edi(invoices, test_mode=test_mode)

        res = {}
        for invoice in invoices:
            builder = self._get_xml_builder(invoice.company_id)
            # For now, the errors are not displayed anywhere, don't want to annoy the user
            xml_content, errors = builder._export_invoice(invoice)

            # DEBUG: send directly to the test platform (the one used by ecosio)
            #response = self.env['account.edi.common']._check_xml_ecosio(invoice, xml_content, builder._export_invoice_ecosio_schematrons())

            attachment_create_vals = {
                'name': builder._export_invoice_filename(invoice),
                'raw': xml_content,
                'mimetype': 'application/xml',
            }
            # we don't want the Factur-X, E-FFF and NLCIUS xml to appear in the attachment of the invoice when confirming it
            # E-FFF and NLCIUS will appear after the pdf is generated, Factur-X will never appear (it's contained in the PDF)
            if self.code not in ['facturx_1_0_05', 'efff_1', 'nlcius_1']:
                attachment_create_vals.update({'res_id': invoice.id, 'res_model': 'account.move'})

            attachment = self.env['ir.attachment'].create(attachment_create_vals)
            res[invoice] = {
                'success': True,
                'attachment': attachment,
            }

        return res

    def _is_embedding_to_invoice_pdf_needed(self):
        # EXTENDS account_edi
        self.ensure_one()

        if self.code == 'facturx_1_0_05':
            return True
        return super()._is_embedding_to_invoice_pdf_needed()

    def _prepare_invoice_report(self, pdf_writer, edi_document):
        # EXTENDS account_edi
        self.ensure_one()
        if self.code != 'facturx_1_0_05':
            return super()._prepare_invoice_report(pdf_writer, edi_document)
        if not edi_document.attachment_id:
            return

        pdf_writer.embed_odoo_attachment(edi_document.attachment_id, subtype='text/xml')
        if not pdf_writer.is_pdfa and str2bool(
                self.env['ir.config_parameter'].sudo().get_param('edi.use_pdfa', 'False')):
            try:
                pdf_writer.convert_to_pdfa()
            except Exception as e:
                _logger.exception("Error while converting to PDF/A: %s", e)
            metadata_template = self.env.ref('account_edi_ubl_cii.account_invoice_pdfa_3_facturx_metadata',
                                             raise_if_not_found=False)
            if metadata_template:
                content = self.env['ir.qweb']._render('account_edi_ubl_cii.account_invoice_pdfa_3_facturx_metadata', {
                    'title': edi_document.move_id.name,
                    'date': fields.Date.context_today(self),
                })
                pdf_writer.add_file_metadata(content)

    ####################################################
    # Import: Account.edi.format override
    ####################################################

    def _create_invoice_from_xml_tree(self, filename, tree, journal=None):
        # EXTENDS account_edi
        self.ensure_one()

        journal = journal or self.env['account.move']._get_default_journal()

        if not self._is_ubl_cii_available(journal.company_id):
            return super()._create_invoice_from_xml_tree(filename, tree, journal=journal)

        # infer the xml builder
        invoice_xml_builder = self._infer_xml_builder_from_tree(tree)

        if invoice_xml_builder is not None:
            invoice = invoice_xml_builder._import_invoice(journal, filename, tree)
            if invoice:
                return invoice

        return super()._create_invoice_from_xml_tree(filename, tree, journal=journal)

    def _update_invoice_from_xml_tree(self, filename, tree, invoice):
        # EXTENDS account_edi
        self.ensure_one()

        if not self._is_ubl_cii_available(invoice.company_id):
            return super()._update_invoice_from_xml_tree(filename, tree, invoice)

        # infer the xml builder
        invoice_xml_builder = self._infer_xml_builder_from_tree(tree)

        if invoice_xml_builder is not None:
            invoice = invoice_xml_builder._import_invoice(invoice.journal_id, filename, tree, invoice)
            if invoice:
                return invoice

        return super()._update_invoice_from_xml_tree(filename, tree, invoice)

```

## File: models\account_edi_xml_cii_facturx.py

```python
# -*- coding: utf-8 -*-
from odoo import models, _
from odoo.tools import DEFAULT_SERVER_DATE_FORMAT, float_repr, is_html_empty, html2plaintext, cleanup_xml_node
from lxml import etree

from datetime import datetime

import logging

_logger = logging.getLogger(__name__)

DEFAULT_FACTURX_DATE_FORMAT = '%Y%m%d'


class AccountEdiXmlCII(models.AbstractModel):
    _name = "account.edi.xml.cii"
    _inherit = 'account.edi.common'
    _description = "Factur-x/XRechnung CII 2.2.0"

    def _export_invoice_filename(self, invoice):
        return "factur-x.xml"

    def _export_invoice_ecosio_schematrons(self):
        return {
            'invoice': 'de.xrechnung:cii:2.2.0',
            'credit_note': 'de.xrechnung:cii:2.2.0',
        }

    def _export_invoice_constraints(self, invoice, vals):
        constraints = self._invoice_constraints_common(invoice)
        constraints.update({
            # [BR-08]-An Invoice shall contain the Seller postal address (BG-5).
            # [BR-09]-The Seller postal address (BG-5) shall contain a Seller country code (BT-40).
            'seller_postal_address': self._check_required_fields(
                vals['record']['company_id']['partner_id']['commercial_partner_id'], 'country_id'
            ),
            # [BR-DE-9] The element "Buyer post code" (BT-53) must be transmitted. (only mandatory in Germany ?)
            'buyer_postal_address': self._check_required_fields(
                vals['record']['commercial_partner_id'], 'zip'
            ),
            # [BR-DE-4] The element "Seller post code" (BT-38) must be transmitted. (only mandatory in Germany ?)
            'seller_post_code': self._check_required_fields(
                vals['record']['company_id']['partner_id']['commercial_partner_id'], 'zip'
            ),
            # [BR-CO-26]-In order for the buyer to automatically identify a supplier, the Seller identifier (BT-29),
            # the Seller legal registration identifier (BT-30) and/or the Seller VAT identifier (BT-31) shall be present.
            'seller_identifier': self._check_required_fields(
                vals['record']['company_id'], ['vat']  # 'siret'
            ),
            # [BR-DE-1] An Invoice must contain information on "PAYMENT INSTRUCTIONS" (BG-16)
            # first check that a partner_bank_id exists, then check that there is an account number
            'seller_payment_instructions_1': self._check_required_fields(
                vals['record'], 'partner_bank_id'
            ),
            'seller_payment_instructions_2': self._check_required_fields(
                vals['record']['partner_bank_id'], 'sanitized_acc_number',
                _("The field 'Sanitized Account Number' is required on the Recipient Bank.")
            ),
            # [BR-DE-6] The element "Seller contact telephone number" (BT-42) must be transmitted.
            'seller_phone': self._check_required_fields(
                vals['record']['company_id']['partner_id']['commercial_partner_id'], ['phone', 'mobile'],
            ),
            # [BR-DE-7] The element "Seller contact email address" (BT-43) must be transmitted.
            'seller_email': self._check_required_fields(
                vals['record']['company_id'], 'email'
            ),
            # [BR-CO-04]-Each Invoice line (BG-25) shall be categorized with an Invoiced item VAT category code (BT-151).
            'tax_invoice_line': self._check_required_tax(vals),
            # [BR-IC-02]-An Invoice that contains an Invoice line (BG-25) where the Invoiced item VAT category code (BT-151)
            # is "Intra-community supply" shall contain the Seller VAT Identifier (BT-31) or the Seller tax representative
            # VAT identifier (BT-63) and the Buyer VAT identifier (BT-48).
            'intracom_seller_vat': self._check_required_fields(vals['record']['company_id'], 'vat') if vals['intracom_delivery'] else None,
            'intracom_buyer_vat': self._check_required_fields(vals['record']['commercial_partner_id'], 'vat') if vals['intracom_delivery'] else None,
            # [BR-IG-05]-In an Invoice line (BG-25) where the Invoiced item VAT category code (BT-151) is "IGIC" the
            # invoiced item VAT rate (BT-152) shall be greater than 0 (zero).
            'igic_tax_rate': self._check_non_0_rate_tax(vals)
                if vals['record']['commercial_partner_id']['country_id']['code'] == 'ES'
                    and vals['record']['commercial_partner_id']['zip']
                    and vals['record']['commercial_partner_id']['zip'][:2] in ['35', '38'] else None,
        })
        return constraints

    def _check_required_tax(self, vals):
        for line_vals in vals['invoice_line_vals_list']:
            line = line_vals['line']
            if not vals['tax_details']['invoice_line_tax_details'][line]['tax_details']:
                return _("You should include at least one tax per invoice line. [BR-CO-04]-Each Invoice line (BG-25) "
                         "shall be categorized with an Invoiced item VAT category code (BT-151).")

    def _check_non_0_rate_tax(self, vals):
        for line_vals in vals['invoice_line_vals_list']:
            tax_rate_list = line_vals['line'].tax_ids.flatten_taxes_hierarchy().mapped("amount")
            if not any([rate > 0 for rate in tax_rate_list]):
                return _("When the Canary Island General Indirect Tax (IGIC) applies, the tax rate on "
                         "each invoice line should be greater than 0.")

    def _get_scheduled_delivery_time(self, invoice):
        # don't create a bridge only to get line.sale_line_ids.order_id.picking_ids.date_done
        # line.sale_line_ids.order_id.picking_ids.scheduled_date or line.sale_line_ids.order_id.commitment_date
        return invoice.invoice_date

    def _get_invoicing_period(self, invoice):
        # get the Invoicing period (BG-14): a list of dates covered by the invoice
        # don't create a bridge to get the date range from the timesheet_ids
        return [invoice.invoice_date]

    def _get_exchanged_document_vals(self, invoice):
        return {
            'id': invoice.name,
            'type_code': '380' if invoice.move_type == 'out_invoice' else '381',
            'issue_date_time': invoice.invoice_date,
            'included_note': html2plaintext(invoice.narration) if invoice.narration else "",
        }

    def _export_invoice_vals(self, invoice):

        def format_date(dt):
            # Format the date in the Factur-x standard.
            dt = dt or datetime.now()
            return dt.strftime(DEFAULT_FACTURX_DATE_FORMAT)

        def format_monetary(number, decimal_places=2):
            # Facturx requires the monetary values to be rounded to 2 decimal values
            return float_repr(number, decimal_places)

        def grouping_key_generator(tax_values):
            tax = tax_values['tax_id']
            grouping_key = {
                **self._get_tax_unece_codes(invoice, tax),
                'amount': tax.amount,
                'amount_type': tax.amount_type,
            }
            # If the tax is fixed, we want to have one group per tax
            # s.t. when the invoice is imported, we can try to guess the fixed taxes
            if tax.amount_type == 'fixed':
                grouping_key['tax_name'] = tax.name
            return grouping_key

        # Validate the structure of the taxes
        self._validate_taxes(invoice)

        # Create file content.
        tax_details = invoice._prepare_edi_tax_details(grouping_key_generator=grouping_key_generator)

        # Fixed Taxes: filter them on the document level, and adapt the totals
        # Fixed taxes are not supposed to be taxes in real live. However, this is the way in Odoo to manage recupel
        # taxes in Belgium. Since only one tax is allowed, the fixed tax is removed from totals of lines but added
        # as an extra charge/allowance.
        fixed_taxes_keys = [k for k in tax_details['tax_details'] if k['amount_type'] == 'fixed']
        for key in fixed_taxes_keys:
            fixed_tax_details = tax_details['tax_details'].pop(key)
            tax_details['tax_amount_currency'] -= fixed_tax_details['tax_amount_currency']
            tax_details['tax_amount'] -= fixed_tax_details['tax_amount']
            tax_details['base_amount_currency'] += fixed_tax_details['tax_amount_currency']
            tax_details['base_amount'] += fixed_tax_details['tax_amount']

        if 'siret' in invoice.company_id._fields and invoice.company_id.siret:
            seller_siret = invoice.company_id.siret
        else:
            seller_siret = invoice.company_id.company_registry

        buyer_siret = False
        if 'siret' in invoice.commercial_partner_id._fields and invoice.commercial_partner_id.siret:
            buyer_siret = invoice.commercial_partner_id.siret
        template_values = {
            **invoice._prepare_edi_vals_to_export(),
            'tax_details': tax_details,
            'format_date': format_date,
            'format_monetary': format_monetary,
            'is_html_empty': is_html_empty,
            'scheduled_delivery_time': self._get_scheduled_delivery_time(invoice),
            'intracom_delivery': False,
            'ExchangedDocument_vals': self._get_exchanged_document_vals(invoice),
            'seller_specified_legal_organization': seller_siret,
            'buyer_specified_legal_organization': buyer_siret,
            'ship_to_trade_party': invoice.partner_shipping_id if 'partner_shipping_id' in invoice._fields and invoice.partner_shipping_id
                else invoice.commercial_partner_id,
            # Chorus Pro fields
            'buyer_reference': invoice.buyer_reference if 'buyer_reference' in invoice._fields
                and invoice.buyer_reference else invoice.commercial_partner_id.ref,
            'purchase_order_reference': invoice.purchase_order_reference if 'purchase_order_reference' in invoice._fields
                and invoice.purchase_order_reference else invoice.ref or invoice.name,
            'contract_reference': invoice.contract_reference if 'contract_reference' in invoice._fields
                and invoice.contract_reference else '',
        }

        # data used for IncludedSupplyChainTradeLineItem / SpecifiedLineTradeSettlement
        for line_vals in template_values['invoice_line_vals_list']:
            line = line_vals['line']
            line_vals['unece_uom_code'] = self._get_uom_unece_code(line)

        # data used for ApplicableHeaderTradeSettlement / ApplicableTradeTax (at the end of the xml)
        for tax_detail_vals in template_values['tax_details']['tax_details'].values():
            # /!\ -0.0 == 0.0 in python but not in XSLT, so it can raise a fatal error when validating the XML
            # if 0.0 is expected and -0.0 is given.
            amount_currency = tax_detail_vals['tax_amount_currency']
            tax_detail_vals['calculated_amount'] = template_values['balance_multiplicator'] * amount_currency \
                if not invoice.currency_id.is_zero(amount_currency) else 0

            if tax_detail_vals.get('tax_category_code') == 'K':
                template_values['intracom_delivery'] = True
            # [BR - IC - 11] - In an Invoice with a VAT breakdown (BG-23) where the VAT category code (BT-118) is
            # "Intra-community supply" the Actual delivery date (BT-72) or the Invoicing period (BG-14) shall not be blank.
            if tax_detail_vals.get('tax_category_code') == 'K' and not template_values['scheduled_delivery_time']:
                date_range = self._get_invoicing_period(invoice)
                template_values['billing_start'] = min(date_range)
                template_values['billing_end'] = max(date_range)

        # One of the difference between XRechnung and Facturx is the following. Submitting a Facturx to XRechnung
        # validator raises a warning, but submitting a XRechnung to Facturx raises an error.
        supplier = invoice.company_id.partner_id.commercial_partner_id
        if supplier.country_id.code == 'DE':
            template_values['document_context_id'] = "urn:cen.eu:en16931:2017#compliant#urn:xoev-de:kosit:standard:xrechnung_2.2"
        else:
            template_values['document_context_id'] = "urn:cen.eu:en16931:2017#conformant#urn:factur-x.eu:1p0:extended"

        # Fixed taxes: add them as charges on the invoice lines
        balance_sign = -1 if invoice.is_inbound() else 1
        for line_vals in template_values['invoice_line_vals_list']:
            line_vals['allowance_charge_vals_list'] = []
            for grouping_key, tax_detail in tax_details['invoice_line_tax_details'][line_vals['line']]['tax_details'].items():
                if grouping_key['amount_type'] == 'fixed':
                    line_vals['allowance_charge_vals_list'].append({
                        'indicator': 'true',
                        'reason': tax_detail['group_tax_details'][0]['tax_id'].name,
                        'reason_code': 'AEO',
                        'amount': balance_sign * tax_detail['tax_amount_currency'],
                    })
            sum_fixed_taxes = sum(x['amount'] for x in line_vals['allowance_charge_vals_list'])
            line_vals['line_total_amount'] = line_vals['line'].price_subtotal + sum_fixed_taxes

        # Fixed taxes: set the total adjusted amounts on the document level
        template_values['tax_basis_total_amount'] = balance_sign * tax_details['base_amount_currency']
        template_values['tax_total_amount'] = balance_sign * tax_details['tax_amount_currency']

        return template_values

    def _export_invoice(self, invoice):
        vals = self._export_invoice_vals(invoice)
        errors = [constraint for constraint in self._export_invoice_constraints(invoice, vals).values() if constraint]
        xml_content = self.env['ir.qweb']._render('account_edi_ubl_cii.account_invoice_facturx_export_22', vals)
        return etree.tostring(cleanup_xml_node(xml_content), xml_declaration=True, encoding='UTF-8'), set(errors)

    # -------------------------------------------------------------------------
    # IMPORT
    # -------------------------------------------------------------------------

    def _import_fill_invoice_form(self, journal, tree, invoice_form, qty_factor):
        logs = []

        if qty_factor == -1:
            logs.append(_("The invoice has been converted into a credit note and the quantities have been reverted."))

        # ==== partner_id ====

        role = invoice_form.journal_id.type == 'purchase' and 'SellerTradeParty' or 'BuyerTradeParty'
        name = self._find_value(f"//ram:{role}/ram:Name", tree)
        mail = self._find_value(f"//ram:{role}//ram:URIID[@schemeID='SMTP']", tree)
        vat = self._find_value(f"//ram:{role}/ram:SpecifiedTaxRegistration/ram:ID", tree)
        phone = self._find_value(f"//ram:{role}/ram:DefinedTradeContact/ram:TelephoneUniversalCommunication/ram:CompleteNumber", tree)
        self._import_retrieve_and_fill_partner(invoice_form, name=name, phone=phone, mail=mail, vat=vat)

        # ==== currency_id ====

        currency_code_node = tree.find('.//{*}InvoiceCurrencyCode')
        if currency_code_node is not None:
            currency = self.env['res.currency'].with_context(active_test=False).search([
                ('name', '=', currency_code_node.text),
            ], limit=1)
            if currency:
                if not currency.active:
                    logs.append(_("The currency '%s' is not active.", currency.name))
                invoice_form.currency_id = currency
            else:
                logs.append(_("Could not retrieve currency: %s. Did you enable the multicurrency option and "
                              "activate the currency ?", currency_code_node.text))

        # ==== Reference ====

        ref_node = tree.find('./{*}ExchangedDocument/{*}ID')
        if ref_node is not None:
            invoice_form.ref = ref_node.text

        # === Note/narration ====

        narration = ""
        note_node = tree.find('./{*}ExchangedDocument/{*}IncludedNote/{*}Content')
        if note_node is not None and note_node.text:
            narration += note_node.text + "\n"

        payment_terms_node = tree.find('.//{*}SpecifiedTradePaymentTerms/{*}Description')
        if payment_terms_node is not None and payment_terms_node.text:
            narration += payment_terms_node.text + "\n"

        invoice_form.narration = narration

        # ==== payment_reference ====

        payment_reference_node = tree.find('.//{*}BuyerOrderReferencedDocument/{*}IssuerAssignedID')
        if payment_reference_node is not None:
            invoice_form.payment_reference = payment_reference_node.text

        # ==== invoice_date ====

        invoice_date_node = tree.find('./{*}ExchangedDocument/{*}IssueDateTime/{*}DateTimeString')
        if invoice_date_node is not None and invoice_date_node.text:
            date_str = invoice_date_node.text.strip()
            date_obj = datetime.strptime(date_str, DEFAULT_FACTURX_DATE_FORMAT)
            invoice_form.invoice_date = date_obj.strftime(DEFAULT_SERVER_DATE_FORMAT)

        # ==== invoice_date_due ====

        invoice_date_due_node = tree.find('.//{*}SpecifiedTradePaymentTerms/{*}DueDateDateTime/{*}DateTimeString')
        if invoice_date_due_node is not None and invoice_date_due_node.text:
            date_str = invoice_date_due_node.text.strip()
            date_obj = datetime.strptime(date_str, DEFAULT_FACTURX_DATE_FORMAT)
            invoice_form.invoice_date_due = date_obj.strftime(DEFAULT_SERVER_DATE_FORMAT)

        # ==== invoice_line_ids: AllowanceCharge (document level) ====

        logs += self._import_fill_invoice_allowance_charge(tree, invoice_form, journal, qty_factor)

        # ==== Prepaid amount ====

        prepaid_node = tree.find('.//{*}ApplicableHeaderTradeSettlement/'
                                 '{*}SpecifiedTradeSettlementHeaderMonetarySummation/{*}TotalPrepaidAmount')
        logs += self._import_log_prepaid_amount(invoice_form, prepaid_node, qty_factor)

        # ==== invoice_line_ids ====

        line_nodes = tree.findall('./{*}SupplyChainTradeTransaction/{*}IncludedSupplyChainTradeLineItem')
        if line_nodes is not None:
            for i, invl_el in enumerate(line_nodes):
                with invoice_form.invoice_line_ids.new() as invoice_line_form:
                    invoice_line_form.sequence = i
                    invl_logs = self._import_fill_invoice_line_form(journal, invl_el, invoice_form, invoice_line_form, qty_factor)
                    logs += invl_logs

        return invoice_form, logs

    def _import_fill_invoice_line_form(self, journal, tree, invoice_form, invoice_line_form, qty_factor):
        logs = []

        # Product.
        name = self._find_value('.//ram:SpecifiedTradeProduct/ram:Name', tree)
        invoice_line_form.product_id = self.env['account.edi.format']._retrieve_product(
            default_code=self._find_value('.//ram:SpecifiedTradeProduct/ram:SellerAssignedID', tree),
            name=name,
            barcode=self._find_value('.//ram:SpecifiedTradeProduct/ram:GlobalID', tree)
        )
        # force original line description instead of the one copied from product's Sales Description
        if name:
            invoice_line_form.name = name

        xpath_dict = {
            'basis_qty': [
                './{*}SpecifiedLineTradeAgreement/{*}GrossPriceProductTradePrice/{*}BasisQuantity',
                './{*}SpecifiedLineTradeAgreement/{*}NetPriceProductTradePrice/{*}BasisQuantity'
            ],
            'gross_price_unit': './{*}SpecifiedLineTradeAgreement/{*}GrossPriceProductTradePrice/{*}ChargeAmount',
            'rebate': './{*}SpecifiedLineTradeAgreement/{*}GrossPriceProductTradePrice/{*}AppliedTradeAllowanceCharge/{*}ActualAmount',
            'net_price_unit': './{*}SpecifiedLineTradeAgreement/{*}NetPriceProductTradePrice/{*}ChargeAmount',
            'billed_qty': './{*}SpecifiedLineTradeDelivery/{*}BilledQuantity',
            'allowance_charge': './/{*}SpecifiedLineTradeSettlement/{*}SpecifiedTradeAllowanceCharge',
            'allowance_charge_indicator': './{*}ChargeIndicator/{*}Indicator',
            'allowance_charge_amount': './{*}ActualAmount',
            'allowance_charge_reason': './{*}Reason',
            'allowance_charge_reason_code': './{*}ReasonCode',
            'line_total_amount': './{*}SpecifiedLineTradeSettlement/{*}SpecifiedTradeSettlementLineMonetarySummation/{*}LineTotalAmount',
        }
        inv_line_vals = self._import_fill_invoice_line_values(tree, xpath_dict, invoice_line_form, qty_factor)
        # retrieve tax nodes
        tax_nodes = tree.findall('.//{*}ApplicableTradeTax/{*}RateApplicablePercent')
        return self._import_fill_invoice_line_taxes(journal, tax_nodes, invoice_line_form, inv_line_vals, logs)

    # -------------------------------------------------------------------------
    # IMPORT : helpers
    # -------------------------------------------------------------------------

    def _get_import_document_amount_sign(self, filename, tree):
        """
        In factur-x, an invoice has code 380 and a credit note has code 381. However, a credit note can be expressed
        as an invoice with negative amounts. For this case, we need a factor to take the opposite of each quantity
        in the invoice.
        """
        move_type_code = tree.find('.//{*}ExchangedDocument/{*}TypeCode')
        if move_type_code is None:
            return None, None
        if move_type_code.text == '381':
            return ('in_refund', 'out_refund'), 1
        if move_type_code.text == '380':
            amount_node = tree.find('.//{*}SpecifiedTradeSettlementHeaderMonetarySummation/{*}TaxBasisTotalAmount')
            if amount_node is not None and float(amount_node.text) < 0:
                return ('in_refund', 'out_refund'), -1
            return ('in_invoice', 'out_invoice'), 1

```

## File: models\account_edi_xml_ubl_20.py

```python
# -*- coding: utf-8 -*-

from odoo import models, _
from odoo.tools import html2plaintext, cleanup_xml_node
from lxml import etree


class AccountEdiXmlUBL20(models.AbstractModel):
    _name = "account.edi.xml.ubl_20"
    _inherit = 'account.edi.common'
    _description = "UBL 2.0"

    # -------------------------------------------------------------------------
    # EXPORT
    # -------------------------------------------------------------------------

    def _export_invoice_filename(self, invoice):
        return f"{invoice.name.replace('/', '_')}_ubl_20.xml"

    def _export_invoice_ecosio_schematrons(self):
        return {
            'invoice': 'org.oasis-open:invoice:2.0',
            'credit_note': 'org.oasis-open:creditnote:2.0',
        }

    def _get_country_vals(self, country):
        return {
            'country': country,

            'identification_code': country.code,
            'name': country.name,
        }

    def _get_partner_party_identification_vals_list(self, partner):
        return []

    def _get_partner_address_vals(self, partner):
        return {
            'street_name': partner.street,
            'additional_street_name': partner.street2,
            'city_name': partner.city,
            'postal_zone': partner.zip,
            'country_subentity': partner.state_id.name,
            'country_subentity_code': partner.state_id.code,
            'country_vals': self._get_country_vals(partner.country_id),
        }

    def _get_partner_party_tax_scheme_vals_list(self, partner, role):
        return [{
            'registration_name': partner.name,
            'company_id': partner.vat,
            'registration_address_vals': self._get_partner_address_vals(partner),
            'TaxScheme_vals': {},
            'tax_scheme_id': 'VAT',
        }]

    def _get_partner_party_legal_entity_vals_list(self, partner):
        commercial_partner = partner.commercial_partner_id

        return [{
            'commercial_partner': commercial_partner,

            'registration_name': commercial_partner.name,
            'company_id': commercial_partner.vat,
            'registration_address_vals': self._get_partner_address_vals(commercial_partner),
        }]

    def _get_partner_contact_vals(self, partner):
        return {
            'id': partner.id,
            'name': partner.name,
            'telephone': partner.phone or partner.mobile,
            'electronic_mail': partner.email,
        }

    def _get_partner_party_vals(self, partner, role):
        return {
            'partner': partner,
            'party_identification_vals': self._get_partner_party_identification_vals_list(partner),
            'party_name_vals': [{'name': partner.name}],
            'postal_address_vals': self._get_partner_address_vals(partner),
            'party_tax_scheme_vals': self._get_partner_party_tax_scheme_vals_list(partner, role),
            'party_legal_entity_vals': self._get_partner_party_legal_entity_vals_list(partner),
            'contact_vals': self._get_partner_contact_vals(partner),
        }

    def _get_invoice_period_vals_list(self, invoice):
        """
        For now, we cannot fill this data from an invoice
        This corresponds to the 'delivery or invoice period'. For UBL Bis 3, in the case of intra-community supply,
        the Actual delivery date (BT-72) or the Invoicing period (BG-14) should be present under the form:
        {
            'start_date': str,
            'end_date': str,
        }.
        """
        return []

    def _get_delivery_vals_list(self, invoice):
        # the data is optional, except for ubl bis3 (see the override, where we need to set a default delivery address)
        if 'partner_shipping_id' in invoice._fields:
            return [{
                'actual_delivery_date': None,
                'delivery_location_vals': {
                    'delivery_address_vals': self._get_partner_address_vals(invoice.partner_shipping_id),
                },
            }]
        else:
            return []

    def _get_bank_address_vals(self, bank):
        return {
            'street_name': bank.street,
            'additional_street_name': bank.street2,
            'city_name': bank.city,
            'postal_zone': bank.zip,
            'country_subentity': bank.state.name,
            'country_subentity_code': bank.state.code,
            'country_vals': self._get_country_vals(bank.country),
        }

    def _get_financial_institution_vals(self, bank):
        return {
            'bank': bank,
            'id': bank.bic,
            'id_attrs': {'schemeID': 'BIC'},
            'name': bank.name,
            'address_vals': self._get_bank_address_vals(bank),
        }

    def _get_financial_institution_branch_vals(self, bank):
        return {
            'bank': bank,
            'id': bank.bic,
            'id_attrs': {'schemeID': 'BIC'},
            'financial_institution_vals': self._get_financial_institution_vals(bank),
        }

    def _get_financial_account_vals(self, partner_bank):
        vals = {
            'bank_account': partner_bank,
            'id': partner_bank.acc_number.replace(' ', ''),
        }

        if partner_bank.bank_id:
            vals['financial_institution_branch_vals'] = self._get_financial_institution_branch_vals(partner_bank.bank_id)

        return vals

    def _get_invoice_payment_means_vals_list(self, invoice):
        vals = {
            'payment_means_code': 30,
            'payment_means_code_attrs': {'name': 'credit transfer'},
            'payment_due_date': invoice.invoice_date_due or invoice.invoice_date,
            'instruction_id': invoice.payment_reference,
            'payment_id_vals': [invoice.payment_reference or invoice.name],
        }

        if invoice.partner_bank_id:
            vals['payee_financial_account_vals'] = self._get_financial_account_vals(invoice.partner_bank_id)

        return [vals]

    def _get_invoice_payment_terms_vals_list(self, invoice):
        payment_term = invoice.invoice_payment_term_id
        if payment_term:
            return [{'note_vals': [payment_term.name]}]
        else:
            return []

    def _get_invoice_tax_totals_vals_list(self, invoice, taxes_vals):
        balance_sign = -1 if invoice.is_inbound() else 1
        tax_totals_vals = {
            'currency': invoice.currency_id,
            'currency_dp': invoice.currency_id.decimal_places,
            'tax_amount': balance_sign * taxes_vals['tax_amount_currency'],
            'tax_subtotal_vals': [],
        }
        for grouping_key, vals in taxes_vals['tax_details'].items():
            if grouping_key['tax_amount_type'] != 'fixed':
                tax_totals_vals['tax_subtotal_vals'].append({
                    'currency': invoice.currency_id,
                    'currency_dp': invoice.currency_id.decimal_places,
                    'taxable_amount': balance_sign * vals['base_amount_currency'],
                    'tax_amount': balance_sign * vals['tax_amount_currency'],
                    'percent': vals['_tax_category_vals_']['percent'],
                    'tax_category_vals': vals['_tax_category_vals_'],
                })
        return [tax_totals_vals]

    def _get_invoice_line_item_vals(self, line, taxes_vals):
        """ Method used to fill the cac:InvoiceLine/cac:Item node.
        It provides information about what the product you are selling.

        :param line:        An invoice line.
        :param taxes_vals:  The tax details for the current invoice line.
        :return:            A python dictionary.

        """
        product = line.product_id
        taxes = line.tax_ids.flatten_taxes_hierarchy().filtered(lambda t: t.amount_type != 'fixed')
        tax_category_vals_list = self._get_tax_category_list(line.move_id, taxes)
        description = line.name and line.name.replace('\n', ', ')

        return {
            # Simple description about what you are selling.
            'description': description,

            # The name of the item.
            'name': product.name,

            # Identifier of the product.
            'sellers_item_identification_vals': {'id': product.code},

            # The main tax applied. Only one is allowed.
            'classified_tax_category_vals': tax_category_vals_list,
        }

    def _get_document_allowance_charge_vals_list(self, invoice):
        """
        https://docs.peppol.eu/poacc/billing/3.0/bis/#_document_level_allowance_or_charge
        """
        return []

    def _get_invoice_line_allowance_vals_list(self, line, tax_values_list=None):
        """ Method used to fill the cac:InvoiceLine>cac:AllowanceCharge node.

        Allowances are distinguished from charges using the ChargeIndicator node with 'false' as value.

        Note that allowance charges do not exist for credit notes in UBL 2.0, so if we apply discount in Odoo
        the net price will not be consistent with the unit price, but we cannot do anything about it

        :param line:    An invoice line.
        :return:        A list of python dictionaries.
        """
        fixed_tax_charge_vals_list = []
        balance_sign = -1 if line.move_id.is_inbound() else 1
        for grouping_key, tax_details in tax_values_list['tax_details'].items():
            if grouping_key['tax_amount_type'] == 'fixed':
                fixed_tax_charge_vals_list.append({
                    'currency_name': line.currency_id.name,
                    'currency_dp': line.currency_id.decimal_places,
                    'charge_indicator': 'true',
                    'allowance_charge_reason_code': 'AEO',
                    'allowance_charge_reason': tax_details['group_tax_details'][0]['tax_id'].name,
                    'amount': balance_sign * tax_details['tax_amount_currency'],
                })

        if not line.discount:
            return fixed_tax_charge_vals_list

        # Price subtotal without discount:
        net_price_subtotal = line.price_subtotal
        # Price subtotal with discount:
        if line.discount == 100.0:
            gross_price_subtotal = 0.0
        else:
            gross_price_subtotal = line.currency_id.round(net_price_subtotal / (1.0 - (line.discount or 0.0) / 100.0))

        allowance_vals = {
            'currency_name': line.currency_id.name,
            'currency_dp': line.currency_id.decimal_places,

            # Must be 'false' since this method is for allowances.
            'charge_indicator': 'false',

            # A reason should be provided. In Odoo, we only manage discounts.
            # Full code list is available here:
            # https://docs.peppol.eu/poacc/billing/3.0/codelist/UNCL5189/
            'allowance_charge_reason_code': 95,

            # The discount should be provided as an amount.
            'amount': gross_price_subtotal - net_price_subtotal,
        }

        return [allowance_vals] + fixed_tax_charge_vals_list

    def _get_invoice_line_price_vals(self, line):
        """ Method used to fill the cac:InvoiceLine/cac:Price node.
        It provides information about the price applied for the goods and services invoiced.

        :param line:    An invoice line.
        :return:        A python dictionary.
        """
        # Price subtotal without discount:
        net_price_subtotal = line.price_subtotal
        # Price subtotal with discount:
        if line.discount == 100.0:
            gross_price_subtotal = 0.0
        else:
            gross_price_subtotal = net_price_subtotal / (1.0 - (line.discount or 0.0) / 100.0)
        # Price subtotal with discount / quantity:
        gross_price_unit = gross_price_subtotal / line.quantity if line.quantity else 0.0

        uom = super()._get_uom_unece_code(line)

        return {
            'currency': line.currency_id,
            'currency_dp': line.currency_id.decimal_places,

            # The price of an item, exclusive of VAT, after subtracting item price discount.
            'price_amount': gross_price_unit,
            'product_price_dp': self.env['decimal.precision'].precision_get('Product Price'),

            # The number of item units to which the price applies.
            # setting to None -> the xml will not comprise the BaseQuantity (it's not mandatory)
            'base_quantity': None,
            'base_quantity_attrs': {'unitCode': uom},
        }

    def _get_invoice_line_vals(self, line, taxes_vals):
        """ Method used to fill the cac:InvoiceLine node.
        It provides information about the invoice line.

        :param line:    An invoice line.
        :return:        A python dictionary.
        """
        allowance_charge_vals_list = self._get_invoice_line_allowance_vals_list(line, taxes_vals)

        uom = super()._get_uom_unece_code(line)
        total_fixed_tax_amount = sum([
            vals['amount']
            for vals in allowance_charge_vals_list
            if vals['allowance_charge_reason_code'] == 'AEO'
        ])
        return {
            'currency': line.currency_id,
            'currency_dp': line.currency_id.decimal_places,
            'invoiced_quantity': line.quantity,
            'invoiced_quantity_attrs': {'unitCode': uom},
            'line_extension_amount': line.price_subtotal + total_fixed_tax_amount,
            'allowance_charge_vals': allowance_charge_vals_list,
            'tax_total_vals': self._get_invoice_tax_totals_vals_list(line.move_id, taxes_vals),
            'item_vals': self._get_invoice_line_item_vals(line, taxes_vals),
            'price_vals': self._get_invoice_line_price_vals(line),
        }

    def _export_invoice_vals(self, invoice):
        def grouping_key_generator(tax_values):
            tax = tax_values['tax_id']
            tax_category_vals = self._get_tax_category_list(invoice, tax)[0]
            grouping_key = {
                'tax_category_id': tax_category_vals['id'],
                'tax_category_percent': tax_category_vals['percent'],
                '_tax_category_vals_': tax_category_vals,
                'tax_amount_type': tax.amount_type,
            }
            # If the tax is fixed, we want to have one group per tax
            # s.t. when the invoice is imported, we can try to guess the fixed taxes
            if tax.amount_type == 'fixed':
                grouping_key['tax_name'] = tax.name
            return grouping_key

        # Validate the structure of the taxes
        self._validate_taxes(invoice)

        # Compute the tax details for the whole invoice and each invoice line separately.
        taxes_vals = invoice._prepare_edi_tax_details(grouping_key_generator=grouping_key_generator)

        # Fixed Taxes: filter them on the document level, and adapt the totals
        # Fixed taxes are not supposed to be taxes in real live. However, this is the way in Odoo to manage recupel
        # taxes in Belgium. Since only one tax is allowed, the fixed tax is removed from totals of lines but added
        # as an extra charge/allowance.
        fixed_taxes_keys = [k for k in taxes_vals['tax_details'] if k['tax_amount_type'] == 'fixed']
        for key in fixed_taxes_keys:
            fixed_tax_details = taxes_vals['tax_details'].pop(key)
            taxes_vals['tax_amount_currency'] -= fixed_tax_details['tax_amount_currency']
            taxes_vals['tax_amount'] -= fixed_tax_details['tax_amount']
            taxes_vals['base_amount_currency'] += fixed_tax_details['tax_amount_currency']
            taxes_vals['base_amount'] += fixed_tax_details['tax_amount']

        # Compute values for invoice lines.
        line_extension_amount = 0.0

        invoice_lines = invoice.invoice_line_ids.filtered(lambda line: not line.display_type)
        document_allowance_charge_vals_list = self._get_document_allowance_charge_vals_list(invoice)
        invoice_line_vals_list = []
        for line_id, line in enumerate(invoice_lines):
            line_taxes_vals = taxes_vals['invoice_line_tax_details'][line]
            line_vals = self._get_invoice_line_vals(line, line_taxes_vals)
            if not line_vals.get('id'):
                line_vals['id'] = line_id + 1
            invoice_line_vals_list.append(line_vals)

            line_extension_amount += line_vals['line_extension_amount']

        # Compute the total allowance/charge amounts.
        allowance_total_amount = 0.0
        for allowance_charge_vals in document_allowance_charge_vals_list:
            if allowance_charge_vals['charge_indicator'] == 'false':
                allowance_total_amount += allowance_charge_vals['amount']

        supplier = invoice.company_id.partner_id.commercial_partner_id
        customer = invoice.commercial_partner_id

        # OrderReference/SalesOrderID (sales_order_id) is optional
        sales_order_id = 'sale_line_ids' in invoice.invoice_line_ids._fields \
                         and ",".join(invoice.invoice_line_ids.sale_line_ids.order_id.mapped('name'))
        # OrderReference/ID (order_reference) is mandatory inside the OrderReference node !
        order_reference = invoice.ref or invoice.name if sales_order_id else invoice.ref

        balance_sign = -1 if invoice.is_inbound() else 1
        vals = {
            'builder': self,
            'invoice': invoice,
            'supplier': supplier,
            'customer': customer,

            'taxes_vals': taxes_vals,

            'format_float': self.format_float,
            'AddressType_template': 'account_edi_ubl_cii.ubl_20_AddressType',
            'ContactType_template': 'account_edi_ubl_cii.ubl_20_ContactType',
            'PartyType_template': 'account_edi_ubl_cii.ubl_20_PartyType',
            'PaymentMeansType_template': 'account_edi_ubl_cii.ubl_20_PaymentMeansType',
            'TaxCategoryType_template': 'account_edi_ubl_cii.ubl_20_TaxCategoryType',
            'TaxTotalType_template': 'account_edi_ubl_cii.ubl_20_TaxTotalType',
            'AllowanceChargeType_template': 'account_edi_ubl_cii.ubl_20_AllowanceChargeType',
            'InvoiceLineType_template': 'account_edi_ubl_cii.ubl_20_InvoiceLineType',
            'InvoiceType_template': 'account_edi_ubl_cii.ubl_20_InvoiceType',

            'vals': {
                'ubl_version_id': 2.0,
                'id': invoice.name,
                'issue_date': invoice.invoice_date,
                'due_date': invoice.invoice_date_due,
                'note_vals': [html2plaintext(invoice.narration)] if invoice.narration else [],
                'order_reference': order_reference,
                'sales_order_id': sales_order_id,
                'accounting_supplier_party_vals': {
                    'party_vals': self._get_partner_party_vals(supplier, role='supplier'),
                },
                'accounting_customer_party_vals': {
                    'party_vals': self._get_partner_party_vals(customer, role='customer'),
                },
                'invoice_period_vals_list': self._get_invoice_period_vals_list(invoice),
                'delivery_vals_list': self._get_delivery_vals_list(invoice),
                'payment_means_vals_list': self._get_invoice_payment_means_vals_list(invoice),
                'payment_terms_vals': self._get_invoice_payment_terms_vals_list(invoice),
                # allowances at the document level, the allowances on invoices (eg. discount) are on invoice_line_vals
                'allowance_charge_vals': document_allowance_charge_vals_list,
                'tax_total_vals': self._get_invoice_tax_totals_vals_list(invoice, taxes_vals),
                'legal_monetary_total_vals': {
                    'currency': invoice.currency_id,
                    'currency_dp': invoice.currency_id.decimal_places,
                    'line_extension_amount': line_extension_amount,
                    'tax_exclusive_amount': balance_sign * taxes_vals['base_amount_currency'],
                    'tax_inclusive_amount': invoice.amount_total,
                    'allowance_total_amount': allowance_total_amount or None,
                    'prepaid_amount': invoice.amount_total - invoice.amount_residual,
                    'payable_amount': invoice.amount_residual,
                },
                'invoice_line_vals': invoice_line_vals_list,
                'currency_dp': invoice.currency_id.decimal_places,  # currency decimal places
            },
        }

        if invoice.move_type == 'out_invoice':
            vals['main_template'] = 'account_edi_ubl_cii.ubl_20_Invoice'
            vals['vals']['invoice_type_code'] = 380
        else:
            vals['main_template'] = 'account_edi_ubl_cii.ubl_20_CreditNote'
            vals['vals']['credit_note_type_code'] = 381

        return vals

    def _export_invoice_constraints(self, invoice, vals):
        constraints = self._invoice_constraints_common(invoice)
        constraints.update({
            'ubl20_supplier_name_required': self._check_required_fields(vals['supplier'], 'name'),
            'ubl20_customer_name_required': self._check_required_fields(vals['customer'], 'name'),
            'ubl20_commercial_customer_name_required': self._check_required_fields(vals['customer'].commercial_partner_id, 'name'),
            'ubl20_invoice_name_required': self._check_required_fields(invoice, 'name'),
            'ubl20_invoice_date_required': self._check_required_fields(invoice, 'invoice_date'),
        })
        return constraints

    def _export_invoice(self, invoice):
        vals = self._export_invoice_vals(invoice)
        errors = [constraint for constraint in self._export_invoice_constraints(invoice, vals).values() if constraint]
        xml_content = self.env['ir.qweb']._render(vals['main_template'], vals)
        return etree.tostring(cleanup_xml_node(xml_content), xml_declaration=True, encoding='UTF-8'), set(errors)

    # -------------------------------------------------------------------------
    # IMPORT
    # -------------------------------------------------------------------------

    def _import_fill_invoice_form(self, journal, tree, invoice_form, qty_factor):
        logs = []

        if qty_factor == -1:
            logs.append(_("The invoice has been converted into a credit note and the quantities have been reverted."))

        # ==== partner_id ====

        role = "Customer" if invoice_form.journal_id.type == 'sale' else "Supplier"
        vat = self._find_value(f'//cac:Accounting{role}Party/cac:Party//cbc:CompanyID', tree)
        phone = self._find_value(f'//cac:Accounting{role}Party/cac:Party//cbc:Telephone', tree)
        mail = self._find_value(f'//cac:Accounting{role}Party/cac:Party//cbc:ElectronicMail', tree)
        name = self._find_value(f'//cac:Accounting{role}Party/cac:Party//cbc:Name', tree)
        self._import_retrieve_and_fill_partner(invoice_form, name=name, phone=phone, mail=mail, vat=vat)

        # ==== currency_id ====

        currency_code_node = tree.find('.//{*}DocumentCurrencyCode')
        if currency_code_node is not None:
            currency = self.env['res.currency'].with_context(active_test=False).search([
                ('name', '=', currency_code_node.text),
            ], limit=1)
            if currency:
                if not currency.active:
                    logs.append(_("The currency '%s' is not active.", currency.name))
                invoice_form.currency_id = currency
            else:
                logs.append(_("Could not retrieve currency: %s. Did you enable the multicurrency option "
                              "and activate the currency ?", currency_code_node.text))

        # ==== Reference ====

        ref_node = tree.find('./{*}ID')
        if ref_node is not None:
            invoice_form.ref = ref_node.text

        # === Note/narration ====

        narration = ""
        note_node = tree.find('./{*}Note')
        if note_node is not None and note_node.text:
            narration += note_node.text + "\n"

        payment_terms_node = tree.find('./{*}PaymentTerms/{*}Note')  # e.g. 'Payment within 10 days, 2% discount'
        if payment_terms_node is not None and payment_terms_node.text:
            narration += payment_terms_node.text + "\n"

        invoice_form.narration = narration

        # ==== payment_reference ====

        payment_reference_node = tree.find('./{*}PaymentMeans/{*}PaymentID')
        if payment_reference_node is not None:
            invoice_form.payment_reference = payment_reference_node.text

        # ==== invoice_date ====

        invoice_date_node = tree.find('./{*}IssueDate')
        if invoice_date_node is not None:
            invoice_form.invoice_date = invoice_date_node.text

        # ==== invoice_date_due ====

        for xpath in ('./{*}DueDate', './/{*}PaymentDueDate'):
            invoice_date_due_node = tree.find(xpath)
            if invoice_date_due_node is not None:
                invoice_form.invoice_date_due = invoice_date_due_node.text
                break

        # ==== invoice_incoterm_id ====

        incoterm_code_node = tree.find('./{*}TransportExecutionTerms/{*}DeliveryTerms/{*}ID')
        if incoterm_code_node is not None:
            incoterm = self.env['account.incoterms'].search([('code', '=', incoterm_code_node.text)], limit=1)
            if incoterm:
                invoice_form.invoice_incoterm_id = incoterm

        # ==== invoice_line_ids: AllowanceCharge (document level) ====

        logs += self._import_fill_invoice_allowance_charge(tree, invoice_form, journal, qty_factor)

        # ==== Prepaid amount ====

        prepaid_node = tree.find('./{*}LegalMonetaryTotal/{*}PrepaidAmount')
        logs += self._import_log_prepaid_amount(invoice_form, prepaid_node, qty_factor)

        # ==== invoice_line_ids: InvoiceLine/CreditNoteLine ====

        invoice_line_tag = 'InvoiceLine' if invoice_form.move_type in ('in_invoice', 'out_invoice') or qty_factor == -1 else 'CreditNoteLine'
        for i, invl_el in enumerate(tree.findall('./{*}' + invoice_line_tag)):
            with invoice_form.invoice_line_ids.new() as invoice_line_form:
                invoice_line_form.sequence = i
                invl_logs = self._import_fill_invoice_line_form(journal, invl_el, invoice_form, invoice_line_form, qty_factor)
                logs += invl_logs

        return invoice_form, logs

    def _import_fill_invoice_line_form(self, journal, tree, invoice_form, invoice_line_form, qty_factor):
        logs = []

        # Product.
        name = self._find_value('./cac:Item/cbc:Name', tree)
        invoice_line_form.product_id = self.env['account.edi.format']._retrieve_product(
            default_code=self._find_value('./cac:Item/cac:SellersItemIdentification/cbc:ID', tree),
            name=name,
            barcode=self._find_value("./cac:Item/cac:StandardItemIdentification/cbc:ID[@schemeID='0160']", tree),
        )
        # force original line description instead of the one copied from product's Sales Description
        if name:
            invoice_line_form.name = name

        # Description
        description_node = tree.find('./{*}Item/{*}Description')
        name_node = tree.find('./{*}Item/{*}Name')
        if description_node is not None:
            invoice_line_form.name = description_node.text
        elif name_node is not None:
            invoice_line_form.name = name_node.text  # Fallback on Name if Description is not found.

        xpath_dict = {
            'basis_qty': [
                './{*}Price/{*}BaseQuantity',
            ],
            'gross_price_unit': './{*}Price/{*}AllowanceCharge/{*}BaseAmount',
            'rebate': './{*}Price/{*}AllowanceCharge/{*}Amount',
            'net_price_unit': './{*}Price/{*}PriceAmount',
            'billed_qty':  './{*}InvoicedQuantity' if invoice_form.move_type in ('in_invoice', 'out_invoice') or qty_factor == -1 else './{*}CreditedQuantity',
            'allowance_charge': './/{*}AllowanceCharge',
            'allowance_charge_indicator': './{*}ChargeIndicator',
            'allowance_charge_amount': './{*}Amount',
            'allowance_charge_reason': './{*}AllowanceChargeReason',
            'allowance_charge_reason_code': './{*}AllowanceChargeReasonCode',
            'line_total_amount': './{*}LineExtensionAmount',
        }
        inv_line_vals = self._import_fill_invoice_line_values(tree, xpath_dict, invoice_line_form, qty_factor)
        # retrieve tax nodes
        tax_nodes = tree.findall('.//{*}Item/{*}ClassifiedTaxCategory/{*}Percent')
        if not tax_nodes:
            for elem in tree.findall('.//{*}TaxTotal'):
                tax_nodes += elem.findall('.//{*}TaxSubtotal/{*}Percent')
        return self._import_fill_invoice_line_taxes(journal, tax_nodes, invoice_line_form, inv_line_vals, logs)

    # -------------------------------------------------------------------------
    # IMPORT : helpers
    # -------------------------------------------------------------------------

    def _get_import_document_amount_sign(self, filename, tree):
        """
        In UBL, an invoice has tag 'Invoice' and a credit note has tag 'CreditNote'. However, a credit note can be
        expressed as an invoice with negative amounts. For this case, we need a factor to take the opposite
        of each quantity in the invoice.
        """
        if tree.tag == '{urn:oasis:names:specification:ubl:schema:xsd:Invoice-2}Invoice':
            amount_node = tree.find('.//{*}LegalMonetaryTotal/{*}TaxExclusiveAmount')
            if amount_node is not None and float(amount_node.text) < 0:
                return ('in_refund', 'out_refund'), -1
            return ('in_invoice', 'out_invoice'), 1
        if tree.tag == '{urn:oasis:names:specification:ubl:schema:xsd:CreditNote-2}CreditNote':
            return ('in_refund', 'out_refund'), 1
        return None, None

```

## File: models\account_edi_xml_ubl_21.py

```python
# -*- coding: utf-8 -*-
from odoo import models


class AccountEdiXmlUBL21(models.AbstractModel):
    _name = "account.edi.xml.ubl_21"
    _inherit = 'account.edi.xml.ubl_20'
    _description = "UBL 2.1"

    # -------------------------------------------------------------------------
    # EXPORT
    # -------------------------------------------------------------------------

    def _export_invoice_filename(self, invoice):
        return f"{invoice.name.replace('/', '_')}_ubl_21.xml"

    def _export_invoice_ecosio_schematrons(self):
        return {
            'invoice': 'org.oasis-open:invoice:2.1',
            'credit_note': 'org.oasis-open:creditnote:2.1',
        }

    def _export_invoice_vals(self, invoice):
        # EXTENDS account.edi.xml.ubl_20
        vals = super()._export_invoice_vals(invoice)

        vals.update({
            'InvoiceType_template': 'account_edi_ubl_cii.ubl_21_InvoiceType',
            'InvoiceLineType_template': 'account_edi_ubl_cii.ubl_21_InvoiceLineType',
        })

        vals['vals'].update({
            'ubl_version_id': 2.1,
            'buyer_reference': invoice.commercial_partner_id.ref,
        })

        return vals

```

## File: models\account_edi_xml_ubl_bis3.py

```python
# -*- coding: utf-8 -*-

from odoo import models, _
from odoo.addons.account_edi_ubl_cii.models.account_edi_common import COUNTRY_EAS

from stdnum.no import mva


class AccountEdiXmlUBLBIS3(models.AbstractModel):
    _name = "account.edi.xml.ubl_bis3"
    _inherit = 'account.edi.xml.ubl_21'
    _description = "UBL BIS Billing 3.0.12"

    """
    * Documentation of EHF Billing 3.0: https://anskaffelser.dev/postaward/g3/
    * EHF 2.0 is no longer used:
      https://anskaffelser.dev/postaward/g2/announcement/2019-11-14-removal-old-invoicing-specifications/
    * Official doc for EHF Billing 3.0 is the OpenPeppol BIS 3 doc +
      https://anskaffelser.dev/postaward/g3/spec/current/billing-3.0/norway/

        "Based on work done in PEPPOL BIS Billing 3.0, Difi has included Norwegian rules in PEPPOL BIS Billing 3.0 and
        does not see a need to implement a different CIUS targeting the Norwegian market. Implementation of EHF Billing
        3.0 is therefore done by implementing PEPPOL BIS Billing 3.0 without extensions or extra rules."

    Thus, EHF 3 and Bis 3 are actually the same format. The specific rules for NO defined in Bis 3 are added in Bis 3.
    """

    # -------------------------------------------------------------------------
    # EXPORT
    # -------------------------------------------------------------------------

    def _export_invoice_filename(self, invoice):
        return f"{invoice.name.replace('/', '_')}_ubl_bis3.xml"

    def _export_invoice_ecosio_schematrons(self):
        return {
            'invoice': 'eu.peppol.bis3:invoice:3.13.0',
            'credit_note': 'eu.peppol.bis3:creditnote:3.13.0',
        }

    def _get_country_vals(self, country):
        # EXTENDS account.edi.xml.ubl_21
        vals = super()._get_country_vals(country)

        vals.pop('name', None)

        return vals

    def _get_partner_party_tax_scheme_vals_list(self, partner, role):
        # EXTENDS account.edi.xml.ubl_21
        vals_list = super()._get_partner_party_tax_scheme_vals_list(partner, role)

        if not partner.vat:
            return []

        for vals in vals_list:
            vals.pop('registration_name', None)
            vals.pop('registration_address_vals', None)

            # /!\ For Australian companies, the ABN is encoded on the VAT field, but doesn't have the 2 digits prefix,
            # causing a validation error
            if partner.country_id.code == "AU" and partner.vat and not partner.vat.upper().startswith("AU"):
                vals['company_id'] = "AU" + partner.vat

            if partner.country_id.code == "LU" and 'l10n_lu_peppol_identifier' in partner._fields and partner.l10n_lu_peppol_identifier:
                vals['company_id'] = partner.l10n_lu_peppol_identifier

        # sources:
        #  https://anskaffelser.dev/postaward/g3/spec/current/billing-3.0/norway/#_applying_foretaksregisteret
        #  https://docs.peppol.eu/poacc/billing/3.0/bis/#national_rules (NO-R-002 (warning))
        if partner.country_id.code == "NO" and role == 'supplier':
            vals_list.append({
                'company_id': "Foretaksregisteret",
                'tax_scheme_id': "TAX",
            })

        return vals_list

    def _get_partner_party_legal_entity_vals_list(self, partner):
        # EXTENDS account.edi.xml.ubl_21
        vals_list = super()._get_partner_party_legal_entity_vals_list(partner)

        for vals in vals_list:
            vals.pop('registration_address_vals', None)
            if partner.country_id.code == 'NL' and 'l10n_nl_oin' in partner._fields:
                endpoint = partner.l10n_nl_oin or partner.l10n_nl_kvk
                scheme = '0190' if partner.l10n_nl_oin else '0106'
                vals.update({
                    'company_id': endpoint,
                    'company_id_attrs': {'schemeID': scheme},
                })
            if partner.country_id.code == "LU" and 'l10n_lu_peppol_identifier' in partner._fields and partner.l10n_lu_peppol_identifier:
                vals['company_id'] = partner.l10n_lu_peppol_identifier
            if partner.country_id.code == 'DK':
                # DK-R-014: For Danish Suppliers it is mandatory to specify schemeID as "0184" (DK CVR-number) when
                # PartyLegalEntity/CompanyID is used for AccountingSupplierParty
                vals['company_id_attrs'] = {'schemeID': '0184'}

        return vals_list

    def _get_partner_contact_vals(self, partner):
        # EXTENDS account.edi.xml.ubl_21
        vals = super()._get_partner_contact_vals(partner)

        vals.pop('id', None)

        return vals

    def _get_partner_party_vals(self, partner, role):
        # EXTENDS account.edi.xml.ubl_21
        vals = super()._get_partner_party_vals(partner, role)

        vals['endpoint_id'] = partner.vat
        vals['endpoint_id_attrs'] = {'schemeID': COUNTRY_EAS.get(partner.country_id.code)}

        if partner.country_id.code == 'NO' and 'l10n_no_bronnoysund_number' in partner._fields:
            vals.update({
                'endpoint_id': partner.l10n_no_bronnoysund_number,
                'endpoint_id_attrs': {'schemeID': '0192'},
            })
        # [BR-NL-1] Dutch supplier registration number ( AccountingSupplierParty/Party/PartyLegalEntity/CompanyID );
        # With a Dutch supplier (NL), SchemeID may only contain 106 (Chamber of Commerce number) or 190 (OIN number).
        # [BR-NL-10] At a Dutch supplier, for a Dutch customer ( AccountingCustomerParty ) the customer registration
        # number must be filled with Chamber of Commerce or OIN. SchemeID may only contain 106 (Chamber of
        # Commerce number) or 190 (OIN number).
        if partner.country_id.code == 'NL' and 'l10n_nl_oin' in partner._fields:
            if partner.l10n_nl_oin:
                vals.update({
                    'endpoint_id': partner.l10n_nl_oin,
                    'endpoint_id_attrs': {'schemeID': '0190'},
                })
            elif partner.l10n_nl_kvk:
                vals.update({
                    'endpoint_id': partner.l10n_nl_kvk,
                    'endpoint_id_attrs': {'schemeID': '0106'},
                })
        if partner.country_id.code == 'SG' and 'l10n_sg_unique_entity_number' in partner._fields:
            vals.update({
                'endpoint_id': partner.l10n_sg_unique_entity_number,
                'endpoint_id_attrs': {'schemeID': '0195'},
            })
        if partner.country_id.code == "LU" and 'l10n_lu_peppol_identifier' in partner._fields and partner.l10n_lu_peppol_identifier:
            vals['endpoint_id'] = partner.l10n_lu_peppol_identifier

        return vals

    def _get_partner_party_identification_vals_list(self, partner):
        # EXTENDS account.edi.xml.ubl_21
        vals = super()._get_partner_party_identification_vals_list(partner)

        if partner.country_id.code == 'NL' and 'l10n_nl_oin' in partner._fields:
            endpoint = partner.l10n_nl_oin or partner.l10n_nl_kvk
            vals.append({
                'id': endpoint,
            })
        return vals

    def _get_delivery_vals_list(self, invoice):
        # EXTENDS account.edi.xml.ubl_21
        supplier = invoice.company_id.partner_id.commercial_partner_id
        customer = invoice.commercial_partner_id

        economic_area = self.env.ref('base.europe').country_ids.mapped('code') + ['NO']
        intracom_delivery = (customer.country_id.code in economic_area
                             and supplier.country_id.code in economic_area
                             and supplier.country_id != customer.country_id)

        if not intracom_delivery:
            return []

        # [BR-IC-12]-In an Invoice with a VAT breakdown (BG-23) where the VAT category code (BT-118) is
        # "Intra-community supply" the Deliver to country code (BT-80) shall not be blank.

        # [BR-IC-11]-In an Invoice with a VAT breakdown (BG-23) where the VAT category code (BT-118) is
        # "Intra-community supply" the Actual delivery date (BT-72) or the Invoicing period (BG-14)
        # shall not be blank.

        if 'partner_shipping_id' in invoice._fields:
            partner_shipping = invoice.partner_shipping_id
        else:
            partner_shipping = customer

        return [{
            'actual_delivery_date': invoice.invoice_date,
            'delivery_location_vals': {
                'delivery_address_vals': self._get_partner_address_vals(partner_shipping),
            },
        }]

    def _get_partner_address_vals(self, partner):
        # EXTENDS account.edi.xml.ubl_21
        vals = super()._get_partner_address_vals(partner)
        # schematron/openpeppol/3.13.0/xslt/CEN-EN16931-UBL.xslt
        # [UBL-CR-225]-A UBL invoice should not include the AccountingCustomerParty Party PostalAddress CountrySubentityCode
        vals.pop('country_subentity_code', None)
        return vals

    def _get_financial_institution_branch_vals(self, bank):
        # EXTENDS account.edi.xml.ubl_21
        vals = super()._get_financial_institution_branch_vals(bank)
        # schematron/openpeppol/3.13.0/xslt/CEN-EN16931-UBL.xslt
        # [UBL-CR-664]-A UBL invoice should not include the FinancialInstitutionBranch FinancialInstitution
        # xpath test: not(//cac:FinancialInstitution)
        vals.pop('id_attrs', None)
        vals.pop('financial_institution_vals', None)
        return vals

    def _get_invoice_payment_means_vals_list(self, invoice):
        # EXTENDS account.edi.xml.ubl_21
        vals_list = super()._get_invoice_payment_means_vals_list(invoice)

        for vals in vals_list:
            vals.pop('payment_due_date', None)
            vals.pop('instruction_id', None)
            if vals.get('payment_id_vals'):
                vals['payment_id_vals'] = vals['payment_id_vals'][:1]

        return vals_list

    def _get_tax_category_list(self, invoice, taxes):
        # EXTENDS account.edi.xml.ubl_21
        vals_list = super()._get_tax_category_list(invoice, taxes)

        for vals in vals_list:
            vals.pop('name')

        return vals_list

    def _get_invoice_tax_totals_vals_list(self, invoice, taxes_vals):
        # EXTENDS account.edi.xml.ubl_21
        vals_list = super()._get_invoice_tax_totals_vals_list(invoice, taxes_vals)

        for vals in vals_list:
            vals['currency_dp'] = 2
            for subtotal_vals in vals.get('tax_subtotal_vals', []):
                subtotal_vals.pop('percent', None)
                subtotal_vals['currency_dp'] = 2

        return vals_list

    def _get_invoice_line_item_vals(self, line, taxes_vals):
        # EXTENDS account.edi.xml.ubl_21
        line_item_vals = super()._get_invoice_line_item_vals(line, taxes_vals)

        for val in line_item_vals['classified_tax_category_vals']:
            # [UBL-CR-601] TaxExemptionReason must not appear in InvoiceLine Item ClassifiedTaxCategory
            # [BR-E-10] TaxExemptionReason must only appear in TaxTotal TaxSubtotal TaxCategory
            val.pop('tax_exemption_reason')

        return line_item_vals

    def _get_invoice_line_allowance_vals_list(self, line, tax_values_list):
        # EXTENDS account.edi.xml.ubl_21
        vals_list = super()._get_invoice_line_allowance_vals_list(line, tax_values_list)

        for vals in vals_list:
            vals['currency_dp'] = 2

        return vals_list

    def _get_invoice_line_vals(self, line, taxes_vals):
        # EXTENDS account.edi.xml.ubl_21
        vals = super()._get_invoice_line_vals(line, taxes_vals)

        vals.pop('tax_total_vals', None)

        vals['currency_dp'] = 2
        vals['price_vals']['currency_dp'] = 2

        return vals

    def _export_invoice_vals(self, invoice):
        # EXTENDS account.edi.xml.ubl_21
        vals = super()._export_invoice_vals(invoice)

        vals['vals'].update({
            'customization_id': 'urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:billing:3.0',
            'profile_id': 'urn:fdc:peppol.eu:2017:poacc:billing:01:1.0',
            'currency_dp': 2,
            'ubl_version_id': None,
        })
        vals['vals']['legal_monetary_total_vals']['currency_dp'] = 2

        # [NL-R-001] For suppliers in the Netherlands, if the document is a creditnote, the document MUST
        # contain an invoice reference (cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID)
        if vals['supplier'].country_id.code == 'NL' and 'refund' in invoice.move_type:
            vals['vals'].update({
                'billing_reference_vals': {
                    'id': invoice.ref,
                    'issue_date': None,
                }
            })

        return vals

    def _export_invoice_constraints(self, invoice, vals):
        # EXTENDS account.edi.xml.ubl_21
        constraints = super()._export_invoice_constraints(invoice, vals)
        constraints.update(
            self._invoice_constraints_peppol_en16931_ubl(invoice, vals)
        )
        constraints.update(
            self._invoice_constraints_cen_en16931_ubl(invoice, vals)
        )

        return constraints

    def _invoice_constraints_cen_en16931_ubl(self, invoice, vals):
        """
        corresponds to the errors raised by ' schematron/openpeppol/3.13.0/xslt/CEN-EN16931-UBL.xslt' for invoices.
        This xslt was obtained by transforming the corresponding sch
        https://docs.peppol.eu/poacc/billing/3.0/files/CEN-EN16931-UBL.sch.
        """
        eu_countries = self.env.ref('base.europe').country_ids
        intracom_delivery = (vals['customer'].country_id in eu_countries
                             and vals['supplier'].country_id in eu_countries
                             and vals['customer'].country_id != vals['supplier'].country_id)

        constraints = {
            # [BR-S-02]-An Invoice that contains an Invoice line (BG-25) where the Invoiced item VAT category code
            # (BT-151) is "Standard rated" shall contain the Seller VAT Identifier (BT-31), the Seller tax registration
            # identifier (BT-32) and/or the Seller tax representative VAT identifier (BT-63).
            # ---
            # [BR-CO-26]-In order for the buyer to automatically identify a supplier, the Seller identifier (BT-29),
            # the Seller legal registration identifier (BT-30) and/or the Seller VAT identifier (BT-31) shall be present.
            'cen_en16931_seller_vat_identifier': self._check_required_fields(
                vals['supplier'], 'vat'  # this check is larger than the rules above
            ),
            # [BR-61]-If the Payment means type code (BT-81) means SEPA credit transfer, Local credit transfer or
            # Non-SEPA international credit transfer, the Payment account identifier (BT-84) shall be present.
            # note: Payment account identifier is <cac:PayeeFinancialAccount>
            # note: no need to check account_number, because it's a required field for a partner_bank
            'cen_en16931_payment_account_identifier': self._check_required_fields(
                invoice, 'partner_bank_id'
            ) if vals['vals']['payment_means_vals_list'][0]['payment_means_code'] in (30, 58) else None,
            # [BR-62]-The Seller electronic address (BT-34) shall have a Scheme identifier.
            # if this fails, it might just be a missing country when mapping the country to the EAS code
            'cen_en16931_seller_EAS': self._check_required_fields(
                vals['vals']['accounting_supplier_party_vals']['party_vals']['endpoint_id_attrs'], 'schemeID',
                _("No Electronic Address Scheme (EAS) could be found for %s.", vals['customer'].name)
            ),
            # [BR-63]-The Buyer electronic address (BT-49) shall have a Scheme identifier.
            # if this fails, it might just be a missing country when mapping the country to the EAS code
            'cen_en16931_buyer_EAS': self._check_required_fields(
                vals['vals']['accounting_customer_party_vals']['party_vals']['endpoint_id_attrs'], 'schemeID',
                _("No Electronic Address Scheme (EAS) could be found for %s.", vals['customer'].name)
            ),
            # [BR-IC-12]-In an Invoice with a VAT breakdown (BG-23) where the VAT category code (BT-118) is
            # "Intra-community supply" the Deliver to country code (BT-80) shall not be blank.
            'cen_en16931_delivery_country_code': self._check_required_fields(
                vals['vals']['delivery_vals_list'][0], 'delivery_location_vals',
                _("For intracommunity supply, the delivery address should be included.")
            ) if intracom_delivery else None,

            # [BR-IC-11]-In an Invoice with a VAT breakdown (BG-23) where the VAT category code (BT-118) is
            # "Intra-community supply" the Actual delivery date (BT-72) or the Invoicing period (BG-14)
            # shall not be blank.
            'cen_en16931_delivery_date_invoicing_period': self._check_required_fields(
                vals['vals']['delivery_vals_list'][0], 'actual_delivery_date',
                _("For intracommunity supply, the actual delivery date or the invoicing period should be included.")
            ) and self._check_required_fields(
                vals['vals']['invoice_period_vals_list'][0], ['start_date', 'end_date'],
                _("For intracommunity supply, the actual delivery date or the invoicing period should be included.")
            ) if intracom_delivery else None,
        }

        for line in invoice.invoice_line_ids.filtered(lambda x: x.display_type not in ('line_note', 'line_section')):
            if len(line.tax_ids.flatten_taxes_hierarchy().filtered(lambda t: t.amount_type != 'fixed')) != 1:
                # [UBL-SR-48]-Invoice lines shall have one and only one classified tax category.
                # /!\ exception: possible to have any number of ecotaxes (fixed tax) with a regular percentage tax
                constraints.update({'cen_en16931_tax_line': _("Each invoice line shall have one and only one tax.")})

        return constraints

    def _invoice_constraints_peppol_en16931_ubl(self, invoice, vals):
        """
        corresponds to the errors raised by 'schematron/openpeppol/3.13.0/xslt/PEPPOL-EN16931-UBL.xslt' for
        invoices in ecosio. This xslt was obtained by transforming the corresponding sch
        https://docs.peppol.eu/poacc/billing/3.0/files/PEPPOL-EN16931-UBL.sch.

        The national rules (https://docs.peppol.eu/poacc/billing/3.0/bis/#national_rules) are included in this file.
        They always refer to the supplier's country.
        """
        constraints = {
            # PEPPOL-EN16931-R020: Seller electronic address MUST be provided
            'peppol_en16931_ubl_seller_endpoint': self._check_required_fields(
                vals['supplier'], 'vat'
            ),
            # PEPPOL-EN16931-R010: Buyer electronic address MUST be provided
            'peppol_en16931_ubl_buyer_endpoint': self._check_required_fields(
                vals['customer'], 'vat'
            ),
            # PEPPOL-EN16931-R003: A buyer reference or purchase order reference MUST be provided.
            'peppol_en16931_ubl_buyer_ref_po_ref':
                "A buyer reference or purchase order reference must be provided." if self._check_required_fields(
                    vals['vals'], 'buyer_reference'
                ) and self._check_required_fields(vals['vals'], 'order_reference') else None,
        }

        if vals['supplier'].country_id.code == 'NL':
            constraints.update({
                # [NL-R-001] For suppliers in the Netherlands, if the document is a creditnote, the document MUST contain
                # an invoice reference (cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID)
                'nl_r_001': self._check_required_fields(invoice, 'ref') if 'refund' in invoice.move_type else '',

                # [NL-R-002] For suppliers in the Netherlands the supplier’s address (cac:AccountingSupplierParty/cac:Party
                # /cac:PostalAddress) MUST contain street name (cbc:StreetName), city (cbc:CityName) and post code (cbc:PostalZone)
                'nl_r_002_street': self._check_required_fields(vals['supplier'], 'street'),
                'nl_r_002_zip': self._check_required_fields(vals['supplier'], 'zip'),
                'nl_r_002_city': self._check_required_fields(vals['supplier'], 'city'),

                # [NL-R-003] For suppliers in the Netherlands, the legal entity identifier MUST be either a
                # KVK or OIN number (schemeID 0106 or 0190)
                'nl_r_003': _(
                    "The supplier %s must have a KVK or OIN number.",
                    vals['supplier'].display_name
                ) if 'l10n_nl_oin' not in vals['supplier']._fields or 'l10n_nl_kvk' not in vals['supplier']._fields else '',

                # [NL-R-007] For suppliers in the Netherlands, the supplier MUST provide a means of payment
                # (cac:PaymentMeans) if the payment is from customer to supplier
                'nl_r_007': self._check_required_fields(invoice, 'partner_bank_id')
            })

            if vals['customer'].country_id.code == 'NL':
                constraints.update({
                    # [NL-R-004] For suppliers in the Netherlands, if the customer is in the Netherlands, the customer
                    # address (cac:AccountingCustomerParty/cac:Party/cac:PostalAddress) MUST contain the street name
                    # (cbc:StreetName), the city (cbc:CityName) and post code (cbc:PostalZone)
                    'nl_r_004_street': self._check_required_fields(vals['customer'], 'street'),
                    'nl_r_004_city': self._check_required_fields(vals['customer'], 'city'),
                    'nl_r_004_zip': self._check_required_fields(vals['customer'], 'zip'),

                    # [NL-R-005] For suppliers in the Netherlands, if the customer is in the Netherlands,
                    # the customer’s legal entity identifier MUST be either a KVK or OIN number (schemeID 0106 or 0190)
                    'nl_r_005': _(
                        "The customer %s must have a KVK or OIN number.",
                        vals['customer'].display_name
                    ) if 'l10n_nl_oin' not in vals['customer']._fields or 'l10n_nl_kvk' not in vals['customer']._fields else '',
                })

        if vals['supplier'].country_id.code == 'NO':
            vat = vals['supplier'].vat
            constraints.update({
                # NO-R-001: For Norwegian suppliers, a VAT number MUST be the country code prefix NO followed by a
                # valid Norwegian organization number (nine numbers) followed by the letters MVA.
                # Note: mva.is_valid("179728982MVA") is True while it lacks the NO prefix
                'no_r_001': _(
                    "The VAT number of the supplier does not seem to be valid. It should be of the form: NO179728982MVA."
                ) if not mva.is_valid(vat) or len(vat) != 14 or vat[:2] != 'NO' or vat[-3:] != 'MVA' else "",

                'no_supplier_bronnoysund': _(
                    "The supplier %s must have a Bronnoysund company registry.",
                    vals['supplier'].display_name
                ) if 'l10n_no_bronnoysund_number' not in vals['supplier']._fields or not vals['supplier'].l10n_no_bronnoysund_number else "",
            })
        if vals['customer'].country_id.code == 'NO':
            constraints.update({
                'no_customer_bronnoysund': _(
                    "The supplier %s must have a Bronnoysund company registry.",
                    vals['customer'].display_name
                ) if 'l10n_no_bronnoysund_number' not in vals['customer']._fields or not vals['customer'].l10n_no_bronnoysund_number else "",
            })

        return constraints

```

## File: models\account_edi_xml_ubl_efff.py

```python
# -*- coding: utf-8 -*-

from odoo import models

import re


class AccountEdiXmlUBLEFFF(models.AbstractModel):
    _inherit = "account.edi.xml.ubl_20"
    _name = 'account.edi.xml.ubl_efff'
    _description = "E-FFF (BE)"

    # -------------------------------------------------------------------------
    # EXPORT
    # -------------------------------------------------------------------------

    def _export_invoice_filename(self, invoice):
        # official naming convention
        vat = invoice.company_id.partner_id.commercial_partner_id.vat
        return 'efff_%s%s%s.xml' % (vat or '', '_' if vat else '', re.sub(r'[\W_]', '', invoice.name))

    def _export_invoice_ecosio_schematrons(self):
        return None

```

## File: models\account_edi_xml_ubl_xrechnung.py

```python
# -*- coding: utf-8 -*-
from odoo import models


class AccountEdiXmlUBLDE(models.AbstractModel):
    _inherit = "account.edi.xml.ubl_bis3"
    _name = 'account.edi.xml.ubl_de'
    _description = "BIS3 DE (XRechnung)"

    # -------------------------------------------------------------------------
    # EXPORT
    # -------------------------------------------------------------------------

    def _export_invoice_filename(self, invoice):
        return f"{invoice.name.replace('/', '_')}_ubl_de.xml"

    def _export_invoice_ecosio_schematrons(self):
        return {
            'invoice': 'de.xrechnung:ubl-invoice:2.2.0',
            'credit_note': 'de.xrechnung:ubl-creditnote:2.2.0',
        }

    def _export_invoice_vals(self, invoice):
        # EXTENDS account.edi.xml.ubl_bis3
        vals = super()._export_invoice_vals(invoice)

        vals['vals'].update({
            'customization_id': 'urn:cen.eu:en16931:2017#compliant#urn:xoev-de:kosit:standard:xrechnung_2.2#conformant#urn:xoev-de:kosit:extension:xrechnung_2.2',
        })

        return vals

    def _export_invoice_constraints(self, invoice, vals):
        # EXTENDS account.edi.xml.ubl_bis3
        constraints = super()._export_invoice_constraints(invoice, vals)

        constraints.update({
            'bis3_de_supplier_telephone_required': self._check_required_fields(vals['supplier'], ['phone', 'mobile']),
            'bis3_de_supplier_electronic_mail_required': self._check_required_fields(vals['supplier'], 'email'),
        })

        return constraints

```

## File: models\ir_actions_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools import cleanup_xml_node

from lxml import etree
import base64
from xml.sax.saxutils import escape, quoteattr


class IrActionsReport(models.Model):
    _inherit = 'ir.actions.report'

    def _postprocess_pdf_report(self, record, buffer):
        """
        EXTENDS account
        Add the pdf report in XML as base64 string.
        """
        result = super()._postprocess_pdf_report(record, buffer)

        if record._name == 'account.move':
            # exclude efff because it's handled by l10n_be_edi
            format_codes = ['ubl_bis3', 'ubl_de', 'nlcius_1']
            edi_attachments = record.edi_document_ids.filtered(lambda d: d.edi_format_id.code in format_codes).attachment_id
            for edi_attachment in edi_attachments:
                old_xml = base64.b64decode(edi_attachment.with_context(bin_size=False).datas, validate=True)
                tree = etree.fromstring(old_xml)
                anchor_elements = tree.xpath("//*[local-name()='AccountingSupplierParty']")
                additional_document_elements = tree.xpath("//*[local-name()='AdditionalDocumentReference']")
                # with this clause, we ensure the xml are only postprocessed once (even when the invoice is reset to
                # draft then validated again)
                if anchor_elements and not additional_document_elements:
                    pdf = base64.b64encode(buffer.getvalue()).decode()
                    pdf_name = '%s.pdf' % record.name.replace('/', '_')
                    to_inject = '''
                        <cac:AdditionalDocumentReference
                            xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                            xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                            xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                            <cbc:ID>%s</cbc:ID>
                            <cac:Attachment>
                                <cbc:EmbeddedDocumentBinaryObject mimeCode="application/pdf" filename=%s>
                                    %s
                                </cbc:EmbeddedDocumentBinaryObject>
                            </cac:Attachment>
                        </cac:AdditionalDocumentReference>
                    ''' % (escape(pdf_name), quoteattr(pdf_name), pdf)

                    anchor_index = tree.index(anchor_elements[0])
                    tree.insert(anchor_index, etree.fromstring(to_inject))
                    new_xml = etree.tostring(cleanup_xml_node(tree), xml_declaration=True, encoding='UTF-8')
                    edi_attachment.write({
                        'res_model': 'account.move',
                        'res_id': record.id,
                        'datas': base64.b64encode(new_xml),
                        'mimetype': 'application/xml',
                    })

        return result

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import account_edi_common
from . import account_edi_format
from . import account_edi_xml_cii_facturx
from . import account_edi_xml_ubl_20
from . import account_edi_xml_ubl_21
from . import account_edi_xml_ubl_bis3
from . import account_edi_xml_ubl_xrechnung
from . import account_edi_xml_ubl_efff
from . import ir_actions_report

```

