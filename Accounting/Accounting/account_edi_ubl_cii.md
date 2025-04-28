# Odoo Module: account_edi_ubl_cii

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models
from . import wizard

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

Allows to export and import formats: E-FFF, UBL Bis 3, EHF3, NLCIUS, Factur-X (CII), XRechnung (UBL).
When generating the PDF on the invoice, the PDF will be embedded inside the xml for all UBL formats. This allows the
receiver to retrieve the PDF with only the xml file. Note that **EHF3 is fully implemented by UBL Bis 3** (`reference
<https://anskaffelser.dev/postaward/g3/spec/current/billing-3.0/norway/#_implementation>`_).

The formats can be chosen from the journal (Journal > Advanced Settings) linked to the invoice.

Note that E-FFF, NLCIUS and XRechnung (UBL) are only available for Belgian, Dutch and German companies,
respectively. UBL Bis 3 is only available for companies which country is present in the `EAS list
<https://docs.peppol.eu/poacc/billing/3.0/codelist/eas/>`_.

Note also that in order for Chorus Pro to automatically detect the "PDF/A-3 (Factur-X)" format, you need to activate
the "Factur-X PDF/A-3" option on the journal. This option will also validate the xml against the Factur-X and Chorus
Pro rules and show the errors.
    """,
    'depends': ['account'],
    'data': [
        'data/cii_22_templates.xml',
        'data/ubl_20_templates.xml',
        'data/ubl_21_templates.xml',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
        'wizard/account_move_send_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

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
                        <ram:LineID t-out="line_vals.get('index')"/>
                    </ram:AssociatedDocumentLineDocument>

                    <!-- Product. -->
                    <ram:SpecifiedTradeProduct>
                        <ram:GlobalID
                            schemeID="0160"
                            t-if="line.product_id and line.product_id.barcode"
                            t-out="line.product_id.barcode"/>
                        <ram:SellerAssignedID
                            t-if="line.product_id and line.product_id.default_code"
                            t-out="line.product_id.default_code"/>
                        <!-- TODO:
                        <ram:Name t-out="line.product_id.name or line.name"/>
                        <ram:Description t-out="line.name" t-if="line.name != line.product_id.name"/>
                        -->
                        <ram:Name t-out="line.name"/>
                    </ram:SpecifiedTradeProduct>

                    <!-- Amounts. -->
                    <ram:SpecifiedLineTradeAgreement>
                        <!-- Line information: unit_price
                        NB: the gross_price_unit should be the price tax excluded!
                        if price_unit = 100 and tax 10% price_include -> then the gross_price_unit is 91
                        -->
                        <ram:GrossPriceProductTradePrice>
                            <ram:ChargeAmount t-out="format_monetary(line_vals['gross_price_total_unit'], 2)"/>
                            <!-- Discount. -->
                            <ram:AppliedTradeAllowanceCharge t-if="line.discount">
                                <ram:ChargeIndicator>
                                    <udt:Indicator t-translation="off">false</udt:Indicator>
                                </ram:ChargeIndicator>
                                <ram:ActualAmount t-out="format_monetary(line_vals['price_discount_unit'], 2)"/>
                            </ram:AppliedTradeAllowanceCharge>
                        </ram:GrossPriceProductTradePrice>
                        <!-- Line unit price, with discount applied -->
                        <ram:NetPriceProductTradePrice>
                            <ram:ChargeAmount
                                t-out="format_monetary(line_vals['price_subtotal_unit'], 2)"/>
                        </ram:NetPriceProductTradePrice>
                    </ram:SpecifiedLineTradeAgreement>

                    <!-- Quantity. -->
                    <ram:SpecifiedLineTradeDelivery>
                        <ram:BilledQuantity
                                t-att-unitCode="line_vals.get('unece_uom_code')"
                                t-out="line_vals.get('quantity')"/>
                    </ram:SpecifiedLineTradeDelivery>

                    <ram:SpecifiedLineTradeSettlement>
                        <t t-foreach="tax_details['tax_details_per_record'][line]['tax_details'].values()"
                           t-as="tax_detail_vals">
                            <ram:ApplicableTradeTax t-if="tax_detail_vals['amount_type'] == 'percent'">
                                <ram:TypeCode t-translation="off">VAT</ram:TypeCode>
                                <ram:CategoryCode t-out="tax_detail_vals['tax_category_code']"/>
                                <ram:RateApplicablePercent t-out="tax_detail_vals['amount']"/>
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
                <ram:Name t-out="partner.display_name"/>
                <ram:SpecifiedLegalOrganization t-if="specified_legal_organization_val">
                    <ram:ID t-att-schemeID="str('0002')"
                            t-out="specified_legal_organization_val"/>
                </ram:SpecifiedLegalOrganization>

                <ram:DefinedTradeContact t-if="not hide_dtc">
                    <ram:PersonName t-out="partner.name"/>
                    <ram:TelephoneUniversalCommunication t-if="partner.phone or partner.mobile">
                        <ram:CompleteNumber t-out="partner.phone or partner.mobile"/>
                    </ram:TelephoneUniversalCommunication>
                    <ram:EmailURIUniversalCommunication t-if="partner.email">
                        <ram:URIID t-out="partner.email"/>
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
                    <ram:PostcodeCode t-out="partner.zip"/>
                    <ram:LineOne t-out="partner.street"/>
                    <ram:LineTwo t-out="partner.street2"/>
                    <ram:CityName t-out="partner.city"/>
                    <ram:CountryID t-out="partner.country_id.code"/>
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
                        <ram:ID t-out="document_context_id"/>
                    </ram:GuidelineSpecifiedDocumentContextParameter>
                </rsm:ExchangedDocumentContext>

                <!-- Document Headers. -->
                <rsm:ExchangedDocument>
                    <ram:ID t-out="ExchangedDocument_vals.get('id')"/>
                    <ram:TypeCode t-out="ExchangedDocument_vals.get('type_code')"/>
                    <ram:IssueDateTime t-if="ExchangedDocument_vals.get('issue_date_time')">
                        <udt:DateTimeString format="102" t-out="format_date(ExchangedDocument_vals.get('issue_date_time'))"/>
                    </ram:IssueDateTime>
                    <ram:IncludedNote>
                        <ram:Content t-out="ExchangedDocument_vals.get('included_note')"/>
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
                        <ram:BuyerReference t-out="buyer_reference"/>
                        <!-- Seller. -->
                        <ram:SellerTradeParty>
                            <!-- Address. -->
                            <t t-call="account_edi_ubl_cii.account_invoice_partner_facturx_export_22">
                                <t t-set="partner" t-value="record.company_id.partner_id"/>
                                <t t-set="specified_legal_organization_val" t-value="seller_specified_legal_organization"/>
                            </t>

                            <!-- VAT. -->
                            <ram:SpecifiedTaxRegistration t-if="record.company_id.vat">
                                <ram:ID schemeID="VA" t-out="record.company_id.vat"/>
                            </ram:SpecifiedTaxRegistration>
                        </ram:SellerTradeParty>

                        <!-- Customer. -->
                        <ram:BuyerTradeParty>
                            <!-- Address. -->
                            <t t-call="account_edi_ubl_cii.account_invoice_partner_facturx_export_22">
                                <t t-set="partner" t-value="record.partner_id"/>
                                <t t-set="specified_legal_organization_val" t-value="buyer_specified_legal_organization"/>
                            </t>

                            <!-- VAT. -->
                            <ram:SpecifiedTaxRegistration t-if="record.commercial_partner_id.vat">
                                <ram:ID schemeID="VA" t-out="record.commercial_partner_id.vat"/>
                            </ram:SpecifiedTaxRegistration>
                        </ram:BuyerTradeParty>

                        <!-- Reference. -->
                        <ram:BuyerOrderReferencedDocument>
                            <!-- "Engagement Juridique" (used by Chorus Pro) -->
                            <ram:IssuerAssignedID t-out="purchase_order_reference"/>
                        </ram:BuyerOrderReferencedDocument>
                        <ram:ContractReferencedDocument>
                            <!-- "Numéro de Marché" (Used by Chorus Pro) -->
                            <ram:IssuerAssignedID t-out="contract_reference"/>
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
                                                    t-out="format_date(scheduled_delivery_time)"/>
                            </ram:OccurrenceDateTime>
                        </ram:ActualDeliverySupplyChainEvent>
                    </ram:ApplicableHeaderTradeDelivery>

                    <!-- Taxes. -->
                    <ram:ApplicableHeaderTradeSettlement>

                        <ram:PaymentReference t-out="record.payment_reference"/>
                        <ram:InvoiceCurrencyCode t-out="currency.name"/>

                        <!-- Bank account. -->
                        <ram:SpecifiedTradeSettlementPaymentMeans t-if="record.partner_bank_id.sanitized_acc_number">
                            <ram:TypeCode>42</ram:TypeCode>
                            <ram:PayeePartyCreditorFinancialAccount>
                                <ram:IBANID
                                    t-if="record.partner_bank_id.acc_type == 'iban'"
                                    t-out="record.partner_bank_id.sanitized_acc_number"/>
                                <ram:ProprietaryID
                                    t-if="record.partner_bank_id.acc_type != 'iban'"
                                    t-out="record.partner_bank_id.sanitized_acc_number"/>
                            </ram:PayeePartyCreditorFinancialAccount>
                        </ram:SpecifiedTradeSettlementPaymentMeans>

                        <!-- Tax Summary. -->
                        <t t-foreach="tax_details.get('tax_details', {}).values()" t-as="tax_detail_vals">
                            <ram:ApplicableTradeTax>
                                <ram:CalculatedAmount
                                    t-out="format_monetary(tax_detail_vals.get('calculated_amount'), 2)"/>
                                <ram:TypeCode t-translation="off">VAT</ram:TypeCode>
                                <ram:ExemptionReason t-out="tax_detail_vals.get('tax_exemption_reason')"/>
                                <ram:BasisAmount
                                    t-out="format_monetary(tax_detail_vals['base_amount_currency'], 2)"/>
                                <ram:CategoryCode t-out="tax_detail_vals['tax_category_code']"/>
                                <ram:ExemptionReasonCode t-out="tax_detail_vals['tax_exemption_reason_code']"/>
                                <!-- 5 = Tax is exigible on the date on the invoice -->
                                <ram:DueDateTypeCode>5</ram:DueDateTypeCode>
                                <ram:RateApplicablePercent
                                    t-if="tax_detail_vals['amount_type'] == 'percent'"
                                    t-out="tax_detail_vals['amount']"/>
                            </ram:ApplicableTradeTax>
                        </t>

                        <!-- Billing Period -->
                        <ram:BillingSpecifiedPeriod t-if="billing_start and billing_end">
                            <ram:StartDateTime>
                                <udt:DateTimeString format="102" t-out="format_date(billing_start)"/>
                            </ram:StartDateTime>
                            <ram:EndDateTime>
                                <udt:DateTimeString format="102" t-out="format_date(billing_end)"/>
                            </ram:EndDateTime>
                        </ram:BillingSpecifiedPeriod>

                        <!-- Payment Term. -->
                        <ram:SpecifiedTradePaymentTerms>
                            <ram:Description t-if="record.invoice_payment_term_id" t-out="record.invoice_payment_term_id.name"/>
                            <ram:DueDateDateTime t-if="record.invoice_date_due">
                                <udt:DateTimeString format="102" t-out="format_date(record.invoice_date_due)"/>
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
                                t-out="format_monetary(record.amount_total, 2)"/>
                            <ram:TotalPrepaidAmount
                                t-out="format_monetary(record.amount_total - record.amount_residual, 2)"/>
                            <ram:DuePayableAmount
                                t-out="format_monetary(record.amount_residual, 2)"/>
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
                                <rdf:li t-att="{'xml:lang': 'x-default'}" t-out="title"/>
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
                        <xmp:CreateDate t-out="date"/>
                        <xmp:ModifyDate t-out="date"/>
                    </rdf:Description>
                    <rdf:Description xmlns:pdfaExtension="http://www.aiim.org/pdfa/ns/extension/"
                                     xmlns:pdfaSchema="http://www.aiim.org/pdfa/ns/schema#"
                                     xmlns:pdfaProperty="http://www.aiim.org/pdfa/ns/property#" rdf:about="">
                        <pdfaExtension:schemas>
                            <rdf:Bag>
                                <rdf:li rdf:parseType="Resource">
                                    <pdfaSchema:schema t-translation="off">Factur-X PDFA Extension Schema</pdfaSchema:schema>
                                    <pdfaSchema:namespaceURI>urn:factur-x:pdfa:CrossIndustryDocument:invoice:1p0#</pdfaSchema:namespaceURI>
                                    <pdfaSchema:prefix>fx</pdfaSchema:prefix>
                                    <pdfaSchema:property>
                                        <rdf:Seq>
                                            <rdf:li rdf:parseType="Resource">
                                                <pdfaProperty:name t-translation="off">DocumentFileName</pdfaProperty:name>
                                                <pdfaProperty:valueType t-translation="off">Text</pdfaProperty:valueType>
                                                <pdfaProperty:category t-translation="off">external</pdfaProperty:category>
                                                <pdfaProperty:description t-translation="off">name of the embedded XML invoice file</pdfaProperty:description>
                                            </rdf:li>
                                            <rdf:li rdf:parseType="Resource">
                                                <pdfaProperty:name t-translation="off">DocumentType</pdfaProperty:name>
                                                <pdfaProperty:valueType t-translation="off">Text</pdfaProperty:valueType>
                                                <pdfaProperty:category t-translation="off">external</pdfaProperty:category>
                                                <pdfaProperty:description t-translation="off">INVOICE</pdfaProperty:description>
                                            </rdf:li>
                                            <rdf:li rdf:parseType="Resource">
                                                <pdfaProperty:name t-translation="off">Version</pdfaProperty:name>
                                                <pdfaProperty:valueType t-translation="off">Text</pdfaProperty:valueType>
                                                <pdfaProperty:category t-translation="off">external</pdfaProperty:category>
                                                <pdfaProperty:description t-translation="off">The actual version of the Factur-X XML schema</pdfaProperty:description>
                                            </rdf:li>
                                            <rdf:li rdf:parseType="Resource">
                                                <pdfaProperty:name t-translation="off">ConformanceLevel</pdfaProperty:name>
                                                <pdfaProperty:valueType t-translation="off">Text</pdfaProperty:valueType>
                                                <pdfaProperty:category t-translation="off">external</pdfaProperty:category>
                                                <pdfaProperty:description t-translation="off">The conformance level of the embedded Factur-X data</pdfaProperty:description>
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
                        <fx:DocumentType t-translation="off">INVOICE</fx:DocumentType>
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
            <cbc:ID t-out="vals.get('id')"/>
            <cbc:AddressTypeCode t-out="vals.get('address_type_code')"/>
            <cbc:AddressFormatCode t-att="vals.get('address_format_code_attrs', {})" t-out="vals.get('address_format_code')"/>
            <cbc:StreetName t-out="vals.get('street_name')"/>
            <cbc:BuildingNumber t-out="vals.get('building_number')"/>
            <cbc:PlotIdentification t-out="vals.get('plot_identification')"/>
            <cbc:CitySubdivisionName t-out="vals.get('city_subdivision_name ')"/>
            <cbc:AdditionalStreetName t-out="vals.get('additional_street_name')"/>
            <cbc:CityName t-out="vals.get('city_name')"/>
            <cbc:PostalZone t-out="vals.get('postal_zone')"/>
            <cbc:CountrySubentity t-out="vals.get('country_subentity')"/>
            <cbc:CountrySubentityCode t-out="vals.get('country_subentity_code')"/>
            <cac:Country>
                <t t-set="country_vals" t-value="vals.get('country_vals', {})"/>
                <cbc:IdentificationCode
                    t-att="country_vals.get('identification_code_attrs', {})"
                    t-out="country_vals.get('identification_code')"/>
                <cbc:Name t-out="country_vals.get('name')"/>
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
                t-out="vals.get('endpoint_id')"/>
            <cac:PartyIdentification t-foreach="vals.get('party_identification_vals', [])" t-as="party_vals">
                <cbc:ID
                    t-att="party_vals.get('id_attrs', {})"
                    t-out="party_vals.get('id')"/>
            </cac:PartyIdentification>
            <cac:PartyName t-foreach="vals.get('party_name_vals', [])" t-as="party_vals">
                <cbc:Name t-out="party_vals.get('name')"/>
            </cac:PartyName>
            <cac:PostalAddress>
                <t t-call="{{AddressType_template}}">
                    <t t-set="vals" t-value="vals.get('postal_address_vals', {})"/>
                </t>
            </cac:PostalAddress>
            <cac:PartyTaxScheme t-foreach="vals.get('party_tax_scheme_vals', [])" t-as="foreach_vals">
                <cbc:RegistrationName t-out="foreach_vals.get('registration_name')"/>
                <cbc:CompanyID
                    t-att="foreach_vals.get('company_id_attrs', {})"
                    t-out="foreach_vals.get('company_id')"/>
                <cac:RegistrationAddress>
                    <t t-call="{{AddressType_template}}">
                        <t t-set="vals" t-value="foreach_vals.get('registration_address_vals', {})"/>
                    </t>
                </cac:RegistrationAddress>
                <cac:TaxScheme>
                    <t t-set="tax_scheme_vals" t-value="foreach_vals.get('tax_scheme_vals', {})"/>
                    <cbc:ID t-att="tax_scheme_vals.get('id_attrs', {})" t-out="tax_scheme_vals.get('id')"/>
                    <cbc:Name t-out="tax_scheme_vals.get('name')"/>
                </cac:TaxScheme>
            </cac:PartyTaxScheme>
            <cac:PartyLegalEntity t-foreach="vals.get('party_legal_entity_vals', [])" t-as="foreach_vals">
                <cbc:RegistrationName t-out="foreach_vals.get('registration_name')"/>
                <cbc:CompanyID
                    t-att="foreach_vals.get('company_id_attrs', {})"
                    t-out="foreach_vals.get('company_id')"/>
                <cac:RegistrationAddress>
                    <t t-call="{{AddressType_template}}">
                        <t t-set="vals" t-value="foreach_vals.get('registration_address_vals', {})"/>
                    </t>
                </cac:RegistrationAddress>
            </cac:PartyLegalEntity>
            <cac:Contact>
                <t t-set="contact_vals" t-value="vals.get('contact_vals', {})"/>
                <cbc:ID t-out="contact_vals.get('id')"/>
                <cbc:Name t-out="contact_vals.get('name')"/>
                <cbc:Telephone t-out="contact_vals.get('telephone')"/>
                <cbc:ElectronicMail t-out="contact_vals.get('electronic_mail')"/>
            </cac:Contact>
            <cac:Person>
                <t t-set="person_vals" t-value="vals.get('person_vals', {})"/>
                <cbc:FirstName t-out="person_vals.get('first_name')"/>
                <cbc:FamilyName t-out="person_vals.get('family_name')"/>
            </cac:Person>
        </t>
    </template>

    <template id="ubl_20_PaymentMeansType">
        <t xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
           xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
           xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:ID t-out="vals.get('id')"/>
            <cbc:PaymentMeansCode
                t-att="vals.get('payment_means_code_attrs', {})"
                t-out="vals.get('payment_means_code')"/>
            <cbc:PaymentDueDate t-out="vals.get('payment_due_date')"/>
            <cbc:InstructionID t-out="vals.get('instruction_id')"/>
            <cbc:PaymentID t-foreach="vals.get('payment_id_vals', [])" t-as="payment_id" t-out="payment_id"/>
            <cac:PayeeFinancialAccount>
                <t t-set="payee_financial_account_vals" t-value="vals.get('payee_financial_account_vals', {})"/>
                <cbc:ID
                    t-att="payee_financial_account_vals.get('id_attrs', {})"
                    t-out="payee_financial_account_vals.get('id')"/>
                <cac:FinancialInstitutionBranch>
                    <t t-set="financial_institution_branch_vals" t-value="payee_financial_account_vals.get('financial_institution_branch_vals', {})"/>
                    <cbc:ID
                        t-att="financial_institution_branch_vals.get('id_attrs', {})"
                        t-out="financial_institution_branch_vals.get('id')"/>
                    <cac:FinancialInstitution>
                        <t t-set="financial_institution_vals" t-value="financial_institution_branch_vals.get('financial_institution_vals', {})"/>
                        <cbc:ID
                            t-att="financial_institution_vals.get('id_attrs', {})"
                            t-out="financial_institution_vals.get('id')"/>
                        <cbc:Name t-out="financial_institution_vals.get('name')"/>
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

    <template id="ubl_20_PaymentTermsType">
        <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
            xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:ID t-out="vals.get('id')"/>
            <cbc:PaymentMeansID t-out="vals.get('payment_means_id')"/>
            <cbc:Note t-foreach="vals.get('note_vals', [])" t-as="note" t-out="note.get('note')"/>
            <cbc:Amount
                t-att-currencyID="vals.get('currency_name')"
                t-out="format_float(vals.get('amount'), vals.get('currency_dp'))"/>
        </t>
    </template>

    <template id="ubl_20_TaxCategoryType">
        <t xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
           xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
           xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:ID
                t-att="vals.get('id_attrs', {})"
                t-out="vals.get('id')"/>
            <cbc:Name t-out="vals.get('name')"/>
            <cbc:Percent t-out="vals.get('percent')"/>
            <cbc:TaxExemptionReasonCode t-out="vals.get('tax_exemption_reason_code')"/>
            <cbc:TaxExemptionReason t-out="vals.get('tax_exemption_reason')"/>
            <cbc:TierRange t-out="vals.get('tier_range')"/>
            <cac:TaxScheme>
                <t t-set="tax_scheme_vals" t-value="vals.get('tax_scheme_vals', {})"/>
                <cbc:ID
                    t-translation="off"
                    t-att="tax_scheme_vals.get('id_attrs', {})"
                    t-out="tax_scheme_vals.get('id')"/>
                <cbc:Name t-translation="off" t-out="tax_scheme_vals.get('name')"/>
                <cbc:TaxTypeCode t-out="tax_scheme_vals.get('tax_type_code')"/>
            </cac:TaxScheme>
        </t>
    </template>

    <template id="ubl_20_TaxTotalType">
        <t xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
           xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
           xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:TaxAmount
                t-att-currencyID="vals['currency'].name"
                t-out="format_float(vals.get('tax_amount'), vals.get('currency_dp'))"/>
            <cac:TaxSubtotal t-foreach="vals.get('tax_subtotal_vals', [])" t-as="foreach_vals">
                <cbc:TaxableAmount
                    t-att-currencyID="foreach_vals['currency'].name"
                    t-out="format_float(foreach_vals.get('taxable_amount'), foreach_vals.get('currency_dp'))"/>
                <cbc:TaxAmount
                    t-att-currencyID="foreach_vals['currency'].name"
                    t-out="format_float(foreach_vals.get('tax_amount'), foreach_vals.get('currency_dp'))"/>
                <cbc:Percent t-out="foreach_vals.get('percent')"/>
                <cbc:BaseUnitMeasure
                    t-att="foreach_vals.get('base_unit_measure_attrs', {})"
                    t-out="foreach_vals.get('base_unit_measure')"/>
                <cac:TaxCategory>
                    <t t-call="{{TaxCategoryType_template}}">
                        <t t-set="vals" t-value="foreach_vals.get('tax_category_vals', {})"/>
                    </t>
                </cac:TaxCategory>
            </cac:TaxSubtotal>
        </t>
    </template>

    <template id="ubl_20_AllowanceChargeType">
        <t xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
           xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
           xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:ChargeIndicator t-out="vals.get('charge_indicator')"/>
            <cbc:AllowanceChargeReasonCode t-out="vals.get('allowance_charge_reason_code')"/>
            <cbc:AllowanceChargeReason t-out="vals.get('allowance_charge_reason')"/>
            <cbc:MultiplierFactorNumeric t-out="vals.get('multiplier_factor')"/>
            <cbc:Amount
                t-att-currencyID="vals.get('currency_name')"
                t-out="format_float(vals.get('amount'), vals.get('currency_dp'))"/>
            <cbc:BaseAmount
                t-att-currencyID="vals.get('currency_name')"
                t-out="format_float(vals.get('base_amount'), vals.get('currency_dp'))"/>
            <cac:TaxCategory t-foreach="vals.get('tax_category_vals', [])" t-as="foreach_vals">
                <t t-call="{{TaxCategoryType_template}}">
                    <t t-set="vals" t-value="foreach_vals"/>
                </t>
            </cac:TaxCategory>
        </t>
    </template>

    <template id="ubl_20_SignatureType">
        <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
           xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:ID t-out="vals.get('id')"/>
            <cac:SignatoryParty>
                <t t-set="sp_vals" t-value="vals.get('signatory_party_vals', {})"/>
                <cac:PartyIdentification>
                    <cbc:ID t-out="sp_vals.get('party_id')"/>
                </cac:PartyIdentification>
                <cac:PartyName>
                    <cbc:Name t-out="sp_vals.get('party_name')"/>
                </cac:PartyName>
            </cac:SignatoryParty>
            <cac:DigitalSignatureAttachment>
                <t t-set="dsa_vals" t-value="vals.get('digital_signature_attachment_vals', {})"/>
                <cac:ExternalReference>
                    <cbc:URI t-out="dsa_vals.get('external_reference_uri')"/>
                </cac:ExternalReference>
            </cac:DigitalSignatureAttachment>
        </t>
    </template>

    <template id="ubl_20_ResponseType">
        <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:ResponseCode t-out="foreach_vals.get('response_code')"/>
            <cbc:Description t-out="foreach_vals.get('description')"/>
        </t>
    </template>

    <template id="ubl_20_DeliveryType">
        <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
           xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:ID t-out="vals.get('delivery_id')"/>
            <cbc:ActualDeliveryDate t-out="vals.get('actual_delivery_date')"/>
            <cac:DeliveryLocation>
                <cac:Address>
                    <t t-call="{{AddressType_template}}">
                        <t t-set="vals" t-value="vals.get('delivery_location_vals', {}).get('delivery_address_vals', {})"/>
                    </t>
                </cac:Address>
            </cac:DeliveryLocation>
            <cac:DeliveryParty>
                <cac:Party>
                    <t t-call="{{PartyType_template}}">
                        <t t-set="vals" t-value="vals.get('delivery_party_vals', {}).get('party_vals', {})"/>
                    </t>
                </cac:Party>
            </cac:DeliveryParty>
        </t>
    </template>

    <template id="ubl_20_MonetaryTotalType">
        <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:LineExtensionAmount
                t-att-currencyID="vals['currency'].name"
                t-out="format_float(vals.get('line_extension_amount'), vals.get('currency_dp'))"/>
            <cbc:TaxExclusiveAmount
                t-att-currencyID="vals['currency'].name"
                t-out="format_float(vals.get('tax_exclusive_amount'), vals.get('currency_dp'))"/>
            <cbc:TaxInclusiveAmount
                t-att-currencyID="vals['currency'].name"
                t-out="format_float(vals.get('tax_inclusive_amount'), vals.get('currency_dp'))"/>
            <cbc:AllowanceTotalAmount
                t-att-currencyID="vals['currency'].name"
                t-out="format_float(vals.get('allowance_total_amount'), vals.get('currency_dp'))"/>
            <cbc:ChargeTotalAmount
                t-att-currencyID="vals['currency'].name"
                t-out="format_float(vals.get('charge_total_amount'), vals.get('currency_dp'))"/>
            <cbc:PrepaidAmount
                t-att-currencyID="vals['currency'].name"
                t-out="format_float(vals.get('prepaid_amount'), vals.get('currency_dp'))"/>
            <cbc:PayableAmount
                t-att-currencyID="vals['currency'].name"
                t-out="format_float(vals.get('payable_amount'), vals.get('currency_dp'))"/>
        </t>
    </template>

    <template id="ubl_20_CommonLineType">
        <t xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
           xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
           xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:ID t-out="vals.get('id')"/>
            <cbc:Note t-out="vals.get('note')"/>
            <cbc:LineExtensionAmount
                t-att-currencyID="vals['currency'].name"
                t-out="format_float(vals.get('line_extension_amount'), vals.get('currency_dp'))"/>
            <cac:PricingReference>
                <t t-set="pr_vals" t-value="vals.get('pricing_reference_vals', {})"/>
                <cac:AlternativeConditionPrice t-foreach="pr_vals.get('alternative_condition_price_vals', [])" t-as="foreach_vals">
                    <cbc:PriceAmount
                        t-att-currencyID="foreach_vals['currency'].name"
                        t-out="format_float(foreach_vals.get('price_amount'), foreach_vals.get('price_amount_dp'))"/>
                    <cbc:PriceTypeCode
                        t-out="foreach_vals.get('price_type_code')"/>
                </cac:AlternativeConditionPrice>
            </cac:PricingReference>
            <cac:TaxTotal t-foreach="vals.get('tax_total_vals', [])" t-as="foreach_vals">
                <t t-call="{{TaxTotalType_template}}">
                    <t t-set="vals" t-value="foreach_vals"/>
                </t>
            </cac:TaxTotal>
            <cac:Item>
                <t t-set="item_vals" t-value="vals.get('item_vals', {})"/>
                <cbc:Description t-out="item_vals.get('description')"/>
                <cbc:Name t-out="item_vals.get('name')"/>
                <cac:SellersItemIdentification>
                    <cbc:ID t-out="item_vals.get('sellers_item_identification_vals', {}).get('id')"/>
                </cac:SellersItemIdentification>
                <cac:CommodityClassification t-foreach="item_vals.get('commodity_classification_vals', [])" t-as="foreach_vals">
                    <cbc:ItemClassificationCode
                            t-att="foreach_vals.get('item_classification_attrs', {})"
                            t-out="foreach_vals.get('item_classification_code')"/>
                </cac:CommodityClassification>
                <cac:ClassifiedTaxCategory t-foreach="item_vals.get('classified_tax_category_vals', [])" t-as="foreach_vals">
                    <t t-call="{{TaxCategoryType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:ClassifiedTaxCategory>
            </cac:Item>
            <cac:Price>
                <t t-set="price_vals" t-value="vals.get('price_vals', {})"/>
                <cbc:PriceAmount
                    t-att-currencyID="price_vals['currency'].name"
                    t-out="price_vals.get('price_amount')"/>
                <!-- nbr of item units to which the price applies), i.e.: 1 Dozen = 12 units, not mandatory -->
                <cbc:BaseQuantity
                    t-att="price_vals.get('base_quantity_attrs', {})"
                    t-out="price_vals.get('base_quantity')"/>
            </cac:Price>
        </t>
    </template>

    <template id="ubl_20_InvoiceLineType" inherit_id="account_edi_ubl_cii.ubl_20_CommonLineType" primary="True"
              xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
        <xpath expr="//*[local-name()='Note']" position="after">
            <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:InvoicedQuantity
                    t-att="vals.get('line_quantity_attrs', {})"
                    t-out="vals.get('line_quantity')"/>
            </t>
        </xpath>
        <xpath expr="//*[local-name()='LineExtensionAmount']" position="after">
            <cac:InvoicePeriod t-foreach="vals.get('invoice_period_vals_list', [])" t-as="invoice_period">
                <t t-call="account_edi_ubl_cii.ubl_20_InvoicePeriod">
                    <t t-set="vals" t-value="invoice_period"/>
                </t>
            </cac:InvoicePeriod>
        </xpath>
        <xpath expr="//*[local-name()='TaxTotal']" position="before">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                <cac:AllowanceCharge t-foreach="vals.get('allowance_charge_vals', [])" t-as="foreach_vals">
                    <t t-call="{{AllowanceChargeType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:AllowanceCharge>
            </t>
        </xpath>
    </template>

    <template id="ubl_20_CreditNoteLineType" inherit_id="account_edi_ubl_cii.ubl_20_CommonLineType" primary="True">
        <xpath expr="//*[local-name()='Note']" position="after">
            <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:CreditedQuantity
                    t-att="vals.get('line_quantity_attrs', {})"
                    t-out="vals.get('line_quantity')"/>
            </t>
        </xpath>
    </template>

    <template id="ubl_20_DebitNoteLineType" inherit_id="account_edi_ubl_cii.ubl_20_CommonLineType" primary="True">
        <xpath expr="//*[local-name()='Note']" position="after">
            <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:DebitedQuantity
                    t-att="vals.get('line_quantity_attrs', {})"
                    t-out="vals.get('line_quantity')"/>
            </t>
        </xpath>
    </template>

    <template id="ubl_20_InvoicePeriod"
              xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
              xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
        <cbc:StartDate t-out="vals.get('start_date')"/>
        <cbc:StartTime t-out="vals.get('start_time')"/>
        <cbc:EndDate t-out="vals.get('end_date')"/>
        <cbc:EndTime t-out="vals.get('end_time')"/>
        <cbc:DurationMeasure t-out="vals.get('duration_measure')"/>
        <cbc:DescriptionCode t-out="vals.get('description_code')"/>
        <cbc:Description t-out="vals.get('description')"/>
    </template>

    <template id="ubl_20_CommonType">
        <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
           xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:UBLVersionID t-out="vals.get('ubl_version_id')"/>
            <cbc:CustomizationID t-out="vals.get('customization_id')"/>
            <cbc:ProfileID
                t-att="vals.get('profile_id_attrs', {})"
                t-out="vals.get('profile_id')"/>
            <cbc:ID t-out="vals.get('id')"/>
            <cbc:CopyIndicator t-out="vals.get('copy_indicator')"/>
            <cbc:UUID t-out="vals.get('uuid')"/>
            <cbc:IssueDate t-out="vals.get('issue_date')"/>
            <cbc:Note t-foreach="vals.get('note_vals', [])" t-as="note_vals"
                t-att="note_vals.get('note_attrs', {})"
                t-out="note_vals.get('note')"/>
            <cbc:DocumentCurrencyCode
                t-att="vals.get('document_currency_code_attrs', {})"
                t-out="invoice.currency_id.name.upper()"/>
            <cbc:TaxCurrencyCode
                t-att="vals.get('tax_currency_code_attrs', {})"
                t-out="vals.get('tax_currency_code')"/>
            <cbc:LineCountNumeric t-out="vals.get('line_count_numeric')"/>
            <cac:InvoicePeriod t-foreach="vals.get('invoice_period_vals_list', [])" t-as="invoice_period">
                <t t-call="account_edi_ubl_cii.ubl_20_InvoicePeriod">
                    <t t-set="vals" t-value="invoice_period"/>
                </t>
            </cac:InvoicePeriod>
            <cac:OrderReference>
                <cbc:ID t-out="vals.get('order_reference')"/>
                <cbc:SalesOrderID t-out="vals.get('sales_order_id')"/>
                <cbc:IssueDate t-out="vals.get('order_issue_date')"/>
            </cac:OrderReference>
            <cac:BillingReference>
                <t t-set="billing_reference_vals" t-value="vals.get('billing_reference_vals', {})"/>
                <cac:InvoiceDocumentReference>
                    <cbc:ID t-out="billing_reference_vals.get('id')"/>
                    <cbc:IssueDate t-out="billing_reference_vals.get('issue_date')"/>
                    <cbc:DocumentTypeCode t-out="billing_reference_vals.get('document_type_code')"/>
                </cac:InvoiceDocumentReference>
            </cac:BillingReference>
            <t t-foreach="vals.get('additional_document_reference_list', [])" t-as="foreach_vals">
                <cac:AdditionalDocumentReference>
                    <cbc:ID t-out="foreach_vals.get('id')"/>
                    <cbc:IssueDate t-out="foreach_vals.get('issue_date')"/>
                    <cbc:DocumentTypeCode t-out="foreach_vals.get('document_type_code')"/>
                    <cbc:DocumentType t-out="foreach_vals.get('document_type')"/>
                    <cbc:DocumentDescription t-out="foreach_vals.get('document_description')"/>
                </cac:AdditionalDocumentReference>
            </t>
            <cac:Signature t-foreach="vals.get('signature_vals', [])" t-as="foreach_vals">
                <t t-call="{{SignatureType_template}}">
                    <t t-set="vals" t-value="foreach_vals"/>
                </t>
            </cac:Signature>
            <cac:AccountingSupplierParty>
                <t t-set="accounting_supplier_vals" t-value="vals.get('accounting_supplier_party_vals', {})"/>
                <cbc:CustomerAssignedAccountID t-out="accounting_supplier_vals.get('customer_assigned_account_id')"/>
                <cac:Party>
                    <t t-call="{{PartyType_template}}">
                        <t t-set="vals" t-value="accounting_supplier_vals.get('party_vals', {})"/>
                    </t>
                </cac:Party>
            </cac:AccountingSupplierParty>
            <cac:AccountingCustomerParty>
                <t t-set="accounting_customer_vals" t-value="vals.get('accounting_customer_party_vals', {})"/>
                <cbc:AdditionalAccountID t-out="accounting_customer_vals.get('additional_account_id')"/>
                <cac:Party>
                    <t t-call="{{PartyType_template}}">
                        <t t-set="vals" t-value="accounting_customer_vals.get('party_vals', {})"/>
                    </t>
                </cac:Party>
            </cac:AccountingCustomerParty>
            <cac:TaxTotal t-foreach="vals.get('tax_total_vals', [])" t-as="foreach_vals">
                <t t-call="{{TaxTotalType_template}}">
                    <t t-set="vals" t-value="foreach_vals"/>
                </t>
            </cac:TaxTotal>
        </t>
    </template>

    <template id="ubl_20_InvoiceType" inherit_id="account_edi_ubl_cii.ubl_20_CommonType" primary="True">
        <xpath expr="//*[local-name()='IssueDate']" position="after">
            <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:InvoiceTypeCode
                    t-att="vals.get('document_type_code_attrs', {})"
                    t-out="vals.get('document_type_code')"/>
            </t>
        </xpath>
        <xpath expr="//*[local-name()='AccountingCustomerParty']" position="after">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
               xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cac:Delivery t-foreach="vals.get('delivery_vals_list', [])" t-as="foreach_vals">
                    <t t-call="{{DeliveryType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:Delivery>
                <cac:PaymentMeans t-foreach="vals.get('payment_means_vals_list', [])" t-as="foreach_vals">
                    <t t-call="{{PaymentMeansType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:PaymentMeans>
                <cac:PaymentTerms t-foreach="vals.get('payment_terms_vals', [])" t-as="foreach_vals">
                    <t t-call="{{PaymentTermsType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:PaymentTerms>
            </t>
        </xpath>
        <xpath expr="//*[local-name()='TaxTotal']" position="before">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                <cac:AllowanceCharge t-foreach="vals.get('allowance_charge_vals', [])" t-as="foreach_vals">
                    <t t-call="{{AllowanceChargeType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:AllowanceCharge>
            </t>
        </xpath>
        <xpath expr="//*[local-name()='TaxTotal']" position="after">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                <cac:LegalMonetaryTotal>
                    <t t-call="{{MonetaryTotalType_template}}">
                        <t t-set="vals" t-value="vals.get('monetary_total_vals', {})"/>
                    </t>
                </cac:LegalMonetaryTotal>
                <cac:InvoiceLine t-foreach="vals.get('line_vals', [])" t-as="foreach_vals">
                    <t t-call="{{InvoiceLineType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:InvoiceLine>
            </t>
        </xpath>
    </template>

    <template id="ubl_20_CreditNoteType" inherit_id="account_edi_ubl_cii.ubl_20_CommonType" primary="True">
        <xpath expr="//*[local-name()='InvoicePeriod']" position="after">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                <cac:DiscrepancyResponse t-foreach="vals.get('discrepancy_response_vals', [])" t-as="foreach_vals">
                    <t t-call="{{ResponseType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:DiscrepancyResponse>
            </t>
        </xpath>
        <xpath expr="//*[local-name()='TaxTotal']" position="before">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                <cac:AllowanceCharge t-foreach="vals.get('allowance_charge_vals', [])" t-as="foreach_vals">
                    <t t-call="{{AllowanceChargeType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:AllowanceCharge>
            </t>
        </xpath>
        <xpath expr="//*[local-name()='TaxTotal']" position="after">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                <cac:LegalMonetaryTotal>
                    <t t-call="{{MonetaryTotalType_template}}">
                        <t t-set="vals" t-value="vals.get('monetary_total_vals', {})"/>
                    </t>
                </cac:LegalMonetaryTotal>
                <cac:CreditNoteLine t-foreach="vals.get('line_vals', [])" t-as="foreach_vals">
                    <t t-call="{{CreditNoteLineType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:CreditNoteLine>
            </t>
        </xpath>
    </template>

    <template id="ubl_20_DebitNoteType" inherit_id="account_edi_ubl_cii.ubl_20_CommonType" primary="True">
        <xpath expr="//*[local-name()='InvoicePeriod']" position="after">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                <cac:DiscrepancyResponse t-foreach="vals.get('discrepancy_response_vals', [])" t-as="foreach_vals">
                    <t t-call="{{ResponseType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:DiscrepancyResponse>
            </t>
        </xpath>
        <xpath expr="//*[local-name()='TaxTotal']" position="after">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                <cac:RequestedMonetaryTotal>
                    <t t-call="{{MonetaryTotalType_template}}">
                        <t t-set="vals" t-value="vals.get('monetary_total_vals', {})"/>
                    </t>
                </cac:RequestedMonetaryTotal>
                <cac:DebitNoteLine t-foreach="vals.get('line_vals', [])" t-as="foreach_vals">
                    <t t-call="{{DebitNoteLineType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:DebitNoteLine>
            </t>
        </xpath>
    </template>

    <template id="ubl_20_Invoice">
        <Invoice
            xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
            xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
            xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <t t-call="{{InvoiceType_template}}"/>
        </Invoice>
    </template>

    <template id="ubl_20_CreditNote">
        <CreditNote
            xmlns="urn:oasis:names:specification:ubl:schema:xsd:CreditNote-2"
            xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
            xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <t t-call="{{CreditNoteType_template}}"/>
        </CreditNote>
    </template>

    <template id="ubl_20_DebitNote">
        <DebitNote
            xmlns="urn:oasis:names:specification:ubl:schema:xsd:DebitNote-2"
            xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
            xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <t t-call="{{DebitNoteType_template}}"/>
        </DebitNote>
    </template>

</odoo>

```

## File: data\ubl_21_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="ubl_21_PaymentTermsType" inherit_id="account_edi_ubl_cii.ubl_20_PaymentTermsType">
        <xpath expr="//*[local-name()='Amount']" position="before">
            <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:PaymentPercent t-out="vals.get('payment_percent')"/>
            </t>
        </xpath>
        <xpath expr="//*[local-name()='Amount']" position="after">
            <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:PaymentDueDate t-out="vals.get('payment_due_date')"/>
            </t>
        </xpath>
    </template>

    <template id="ubl_21_CreditNoteLineType" inherit_id="account_edi_ubl_cii.ubl_20_CreditNoteLineType" primary="True">
        <xpath expr="//*[local-name()='TaxTotal']" position="after">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                <cac:AllowanceCharge t-foreach="vals.get('allowance_charge_vals', [])" t-as="foreach_vals">
                    <t t-call="{{AllowanceChargeType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:AllowanceCharge>
            </t>
        </xpath>
    </template>

    <template id="ubl_21_DebitNoteLineType" inherit_id="account_edi_ubl_cii.ubl_20_DebitNoteLineType" primary="True">
        <xpath expr="//*[local-name()='TaxTotal']" position="after">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                <cac:AllowanceCharge t-foreach="vals.get('allowance_charge_vals', [])" t-as="foreach_vals">
                    <t t-call="{{AllowanceChargeType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:AllowanceCharge>
            </t>
        </xpath>
    </template>

    <template id="ubl_21_InvoiceType" inherit_id="account_edi_ubl_cii.ubl_20_InvoiceType" primary="True">
        <xpath expr="//*[local-name()='IssueDate']" position="after">
            <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:DueDate t-out="vals.get('due_date')"/>
            </t>
        </xpath>
        <xpath expr="//*[local-name()='TaxCurrencyCode']" position="after">
            <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:BuyerReference t-out="vals.get('buyer_reference')"/>
            </t>
        </xpath>
    </template>

    <template id="ubl_21_CreditNoteType" inherit_id="account_edi_ubl_cii.ubl_20_CreditNoteType" primary="True">
        <xpath expr="//*[local-name()='IssueDate']" position="after">
            <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:CreditNoteTypeCode
                    t-att="vals.get('document_type_code_attrs', {})"
                    t-out="vals.get('document_type_code')"/>
            </t>
        </xpath>
        <xpath expr="//*[local-name()='TaxCurrencyCode']" position="after">
            <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:BuyerReference t-out="vals.get('buyer_reference')"/>
            </t>
        </xpath>
        <xpath expr="//*[local-name()='TaxTotal']" position="before">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                <cac:Delivery t-foreach="vals.get('delivery_vals_list', [])" t-as="foreach_vals">
                    <t t-call="{{DeliveryType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:Delivery>
                <cac:PaymentMeans t-foreach="vals.get('payment_means_vals_list', [])" t-as="foreach_vals">
                    <t t-call="{{PaymentMeansType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:PaymentMeans>
                <cac:PaymentTerms t-foreach="vals.get('payment_terms_vals', [])" t-as="foreach_vals">
                    <t t-call="{{PaymentTermsType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:PaymentTerms>
            </t>
        </xpath>
    </template>

    <template id="ubl_21_DebitNoteType" inherit_id="account_edi_ubl_cii.ubl_20_DebitNoteType" primary="True">
        <xpath expr="//*[local-name()='TaxCurrencyCode']" position="after">
            <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:BuyerReference t-out="vals.get('buyer_reference')"/>
            </t>
        </xpath>
        <xpath expr="//*[local-name()='TaxTotal']" position="before">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                <cac:AllowanceCharge t-foreach="vals.get('allowance_charge_vals', [])" t-as="foreach_vals">
                    <t t-call="{{AllowanceChargeType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:AllowanceCharge>
                <cac:Delivery t-foreach="vals.get('delivery_vals_list', [])" t-as="foreach_vals">
                    <t t-call="{{DeliveryType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:Delivery>
                <cac:PaymentMeans t-foreach="vals.get('payment_means_vals_list', [])" t-as="foreach_vals">
                    <t t-call="{{PaymentMeansType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:PaymentMeans>
                <cac:PaymentTerms t-foreach="vals.get('payment_terms_vals', [])" t-as="foreach_vals">
                    <t t-call="{{PaymentTermsType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:PaymentTerms>
            </t>
        </xpath>
    </template>

</odoo>

```

## File: models\account_edi_common.py

```python
from odoo import _, models, Command
from odoo.addons.base.models.res_bank import sanitize_account_number
from odoo.exceptions import UserError, ValidationError
from odoo.tools import float_repr, find_xml_value
from odoo.tools.float_utils import float_round
from odoo.tools.misc import clean_context, formatLang
from odoo.tools.zeep import Client

from markupsafe import Markup

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
    'uom.uom_square_meter': 'MTK',
    'uom.uom_square_foot': 'FTK',
    'uom.product_uom_yard': 'YRD',
    'uom.product_uom_millimeter': 'MMT',
}

# -------------------------------------------------------------------------
# ELECTRONIC ADDRESS SCHEME (EAS), see https://docs.peppol.eu/poacc/billing/3.0/codelist/eas/
# -------------------------------------------------------------------------
EAS_MAPPING = {
    'AD': {'9922': 'vat'},
    'AL': {'9923': 'vat'},
    'AT': {'9915': 'vat'},
    'AU': {'0151': 'vat'},
    'BA': {'9924': 'vat'},
    'BE': {'0208': 'company_registry'},
    'BG': {'9926': 'vat'},
    'CH': {'9927': 'vat'},
    'CY': {'9928': 'vat'},
    'CZ': {'9929': 'vat'},
    'DE': {'9930': 'vat'},
    'DK': {'0184': 'vat', '0198': 'vat'},
    'EE': {'9931': 'vat'},
    'ES': {'9920': 'vat'},
    'FI': {'0216': None},
    'FR': {'0009': 'siret', '9957': 'vat'},
    'SG': {'0195': 'l10n_sg_unique_entity_number'},
    'GB': {'9932': 'vat'},
    'GR': {'9933': 'vat'},
    'HR': {'9934': 'vat'},
    'HU': {'9910': 'l10n_hu_eu_vat'},
    'IE': {'9935': 'vat'},
    'IS': {'0196': 'vat'},
    'IT': {'0211': 'vat', '0210': 'l10n_it_codice_fiscale'},
    'JP': {'0221': 'vat'},
    'LI': {'9936': 'vat'},
    'LT': {'9937': 'vat'},
    'LU': {'9938': 'vat'},
    'LV': {'9939': 'vat'},
    'MC': {'9940': 'vat'},
    'ME': {'9941': 'vat'},
    'MK': {'9942': 'vat'},
    'MT': {'9943': 'vat'},
    # Do not add the vat for NL, since: "[NL-R-003] For suppliers in the Netherlands, the legal entity identifier
    # MUST be either a KVK or OIN number (schemeID 0106 or 0190)" in the Bis 3 rules (in PartyLegalEntity/CompanyID).
    'NL': {'0106': None, '0190': None},
    'NO': {'0192': 'l10n_no_bronnoysund_number'},
    'NZ': {'0088': 'company_registry'},
    'PL': {'9945': 'vat'},
    'PT': {'9946': 'vat'},
    'RO': {'9947': 'vat'},
    'RS': {'9948': 'vat'},
    'SE': {'0007': 'company_registry'},
    'SI': {'9949': 'vat'},
    'SK': {'9950': 'vat'},
    'SM': {'9951': 'vat'},
    'TR': {'9952': 'vat'},
    'VA': {'9953': 'vat'},
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

    def _get_currency_decimal_places(self, currency_id):
        # Allows other documents to easily override in case there is a flat max precision number
        return currency_id.decimal_places

    def _get_uom_unece_code(self, line):
        """
        list of codes: https://docs.peppol.eu/poacc/billing/3.0/codelist/UNECERec20/
        or https://unece.org/fileadmin/DAM/cefact/recommendations/bkup_htm/add2c.htm (sorted by letter)
        """
        xmlid = line.product_uom_id.get_external_id()
        if xmlid and line.product_uom_id.id in xmlid:
            return UOM_TO_UNECE_CODE.get(xmlid[line.product_uom_id.id], 'C62')
        return 'C62'

    def _find_value(self, xpath, tree, nsmap=False):
        # avoid 'TypeError: empty namespace prefix is not supported in XPath'
        nsmap = nsmap or {k: v for k, v in tree.nsmap.items() if k is not None}
        return find_xml_value(xpath, tree, nsmap)

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

        if supplier.country_id == customer.country_id:
            if not tax or tax.amount == 0:
                # in theory, you should indicate the precise law article
                return create_dict(tax_category_code='E', tax_exemption_reason=_('Articles 226 items 11 to 15 Directive 2006/112/EN'))
            else:
                return create_dict(tax_category_code='S')  # standard VAT

        if supplier.country_id.code in european_economic_area and supplier.vat:
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
                'tax_scheme_vals': {'id': 'VAT'},
                **tax_unece_codes,
            })
        return res

    # -------------------------------------------------------------------------
    # CONSTRAINTS
    # -------------------------------------------------------------------------

    def _check_required_fields(self, record, field_names, custom_warning_message=""):
        """Check if at least one of the field_names are set on the record/dict

        :param record: either a recordSet or a dict
        :param field_names: The field name or list of field name that has to
                            be checked. If a list is provided, check that at
                            least one of them is set.
        :return: an Error message or None
        """
        if not record:
            return custom_warning_message or _("The element %s is required on %s.", record, ', '.join(field_names))

        if not isinstance(field_names, (list, tuple)):
            field_names = (field_names,)

        has_values = any((field_name in record and record[field_name]) for field_name in field_names)
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
        for line in invoice.invoice_line_ids.filtered(lambda x: x.display_type not in ('line_note', 'line_section') and x._check_edi_line_tax_required()):
            if not line.tax_ids:
                return {'tax_on_line': _("Each invoice line should have at least one tax.")}
        return {}

    # -------------------------------------------------------------------------
    # Import invoice
    # -------------------------------------------------------------------------

    def _import_invoice_ubl_cii(self, invoice, file_data, new=False):
        tree = file_data['xml_tree']

        # Not able to decode the move_type from the xml.
        move_type, qty_factor = self._get_import_document_amount_sign(tree)
        if not move_type:
            return

        # Check for inconsistent move_type.
        journal = invoice.journal_id
        if journal.type == 'sale':
            move_type = 'out_' + move_type
        elif journal.type == 'purchase':
            move_type = 'in_' + move_type
        else:
            return
        if not new and invoice.move_type != move_type:
            # with an email alias to create account_move, first the move is created (using alias_defaults, which
            # contains move_type = 'out_invoice') then the attachment is decoded, if it represents a credit note,
            # the move type needs to be changed to 'out_refund'
            types = {move_type, invoice.move_type}
            if types == {'out_invoice', 'out_refund'} or types == {'in_invoice', 'in_refund'}:
                invoice.move_type = move_type
            else:
                return

        # Update the invoice.
        invoice.move_type = move_type
        with invoice._get_edi_creation() as invoice:
            logs = self._import_fill_invoice_form(invoice, tree, qty_factor)
        if invoice:
            body = Markup("<strong>%s</strong>") % \
                _("Format used to import the invoice: %s",
                  self.env['ir.model']._get(self._name).name)

            if logs:
                body += Markup("<ul>%s</ul>") % \
                    Markup().join(Markup("<li>%s</li>") % l for l in logs)

            invoice.message_post(body=body)

        # For UBL, we should override the computed tax amount if it is less than 0.05 different of the one in the xml.
        # In order to support use case where the tax total is adapted for rounding purpose.
        # This has to be done after the first import in order to let Odoo compute the taxes before overriding if needed.
        with invoice._get_edi_creation() as invoice:
            self._correct_invoice_tax_amount(tree, invoice)

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

        return True

    def _import_retrieve_and_fill_partner(self, invoice, name, phone, mail, vat, country_code=False, peppol_eas=False, peppol_endpoint=False, street=False, street2=False, city=False, zip_code=False):
        """ Retrieve the partner, if no matching partner is found, create it (only if he has a vat and a name) """
        if peppol_eas and peppol_endpoint:
            domain = [('peppol_eas', '=', peppol_eas), ('peppol_endpoint', '=', peppol_endpoint)]
        else:
            domain = False
        invoice.partner_id = self.env['res.partner'] \
            .with_company(invoice.company_id) \
            ._retrieve_partner(name=name, phone=phone, mail=mail, vat=vat, domain=domain)
        if not invoice.partner_id and name and vat:
            partner_vals = {'name': name, 'email': mail, 'phone': phone, 'street': street, 'street2': street2, 'zip': zip_code, 'city': city}
            if peppol_eas and peppol_endpoint:
                partner_vals.update({'peppol_eas': peppol_eas, 'peppol_endpoint': peppol_endpoint})
            country = self.env.ref(f'base.{country_code.lower()}', raise_if_not_found=False) if country_code else False
            if country:
                partner_vals['country_id'] = country.id
            invoice.partner_id = self.env['res.partner'].create(partner_vals)
            if vat and self.env['res.partner']._run_vat_test(vat, country, invoice.partner_id.is_company):
                invoice.partner_id.vat = vat

    def _import_retrieve_and_fill_partner_bank_details(self, invoice, bank_details):
        """ Retrieve the bank account, if no matching bank account is found, create it
        """

        # clear the context, because creation of partner when importing should not depend on the context default values
        ResPartnerBank = self.env['res.partner.bank'].with_env(self.env(context=clean_context(self.env.context)))
        bank_details = list(map(sanitize_account_number, bank_details))

        if invoice.move_type in ('out_refund', 'in_invoice'):
            partner = invoice.partner_id
        elif invoice.move_type in ('out_invoice', 'in_refund'):
            partner = self.env.company.partner_id
        else:
            return

        banks_to_create = []
        acc_number_partner_bank_dict = {
            bank.sanitized_acc_number: bank
            for bank in ResPartnerBank.search(
                [('company_id', 'in', [False, invoice.company_id.id]), ('acc_number', 'in', bank_details)]
            )
        }

        for account_number in bank_details:
            partner_bank = acc_number_partner_bank_dict.get(account_number, ResPartnerBank)

            if partner_bank.partner_id == partner:
                invoice.partner_bank_id = partner_bank
                return
            elif not partner_bank and account_number:
                banks_to_create.append({
                    'acc_number': account_number,
                    'partner_id': partner.id,
                })

        if banks_to_create:
            invoice.partner_bank_id = ResPartnerBank.create(banks_to_create)[0]

    def _import_fill_invoice_allowance_charge(self, tree, invoice, qty_factor):
        logs = []
        if '{urn:oasis:names:specification:ubl:schema:xsd' in tree.tag:
            is_ubl = True
        elif '{urn:un:unece:uncefact:data:standard:' in tree.tag:
            is_ubl = False
        else:
            return

        xpath = './{*}AllowanceCharge' if is_ubl else './{*}SupplyChainTradeTransaction/{*}ApplicableHeaderTradeSettlement/{*}SpecifiedTradeAllowanceCharge'
        allowance_charge_nodes = tree.findall(xpath)
        line_vals = []
        for allow_el in allowance_charge_nodes:
            # get the charge factor
            charge_factor = -1  # factor is -1 for discount, 1 for charge
            if is_ubl:
                charge_indicator_node = allow_el.find('./{*}ChargeIndicator')
            else:
                charge_indicator_node = allow_el.find('./{*}ChargeIndicator/{*}Indicator')
            if charge_indicator_node is not None:
                charge_factor = -1 if charge_indicator_node.text == 'false' else 1

            # get the name
            name = ""
            reason_node = allow_el.find('./{*}AllowanceChargeReason' if is_ubl else './{*}Reason')
            if reason_node is not None:
                name = reason_node.text

            # get quantity and price unit
            quantity = 1
            price_unit = 0
            amount_node = allow_el.find('./{*}Amount' if is_ubl else './{*}ActualAmount')
            base_amount_node = allow_el.find('./{*}BaseAmount' if is_ubl else './{*}BasisAmount')
            # Since there is no quantity associated for the allowance/charge on document level,
            # if we have an invoice with negative amounts, the price was multiplied by -1 and not the quantity
            # See the file in test_files: 'base-negative-inv-correction.xml' VS 'base-example.xml' for 'Insurance'
            if base_amount_node is not None:
                price_unit = float(base_amount_node.text) * charge_factor * qty_factor
                percent_node = allow_el.find('./{*}MultiplierFactorNumeric' if is_ubl else './{*}CalculationPercent')
                if percent_node is not None:
                    quantity = float(percent_node.text) / 100
            elif amount_node is not None:
                price_unit = float(amount_node.text) * charge_factor * qty_factor

            # get taxes
            tax_xpath = './{*}TaxCategory/{*}Percent' if is_ubl else './{*}CategoryTradeTax/{*}RateApplicablePercent'
            tax_ids = []
            for tax_categ_percent_el in allow_el.findall(tax_xpath):
                tax = self.env['account.tax'].search([
                    *self.env['account.tax']._check_company_domain(invoice.company_id),
                    ('amount', '=', float(tax_categ_percent_el.text)),
                    ('amount_type', '=', 'percent'),
                    ('type_tax_use', '=', invoice.journal_id.type),  # Journal type is ensured by _create_invoice_from_xml_tree to be either 'sale' or 'purchase'
                ], limit=1)
                if tax:
                    tax_ids += tax.ids
                else:
                    logs.append(
                        _("Could not retrieve the tax: %s %% for line '%s'.",
                            float(tax_categ_percent_el.text),
                            name)
                    )

            line_vals += [Command.create({
                'sequence': 0,  # be sure to put these lines above the 'real' invoice lines
                'name': name,
                'quantity': quantity,
                'price_unit': price_unit,
                'tax_ids': [Command.set(tax_ids)],
            })]

        invoice.write({'invoice_line_ids': line_vals})
        return logs

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

    def _import_fill_invoice_line_values(self, tree, xpath_dict, invoice_line, qty_factor):
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
        :params: invoice_line
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
                        'tax_amount': float(amount.text) / billed_qty,
                    })
                else:
                    allow_charge_amount += float(amount.text) * discount_factor

        # line_net_subtotal (mandatory)
        price_subtotal = None
        line_total_amount_node = tree.find(xpath_dict['line_total_amount'])
        if line_total_amount_node is not None:
            price_subtotal = float(line_total_amount_node.text)

        ####################################################
        # Setting the values on the invoice_line
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
        amount_fixed_taxes = sum(d['tax_amount'] * billed_qty for d in fixed_taxes_list)
        if billed_qty * price_unit != 0 and price_subtotal is not None:
            discount = 100 * (1 - (price_subtotal - amount_fixed_taxes) / (billed_qty * price_unit))

        # Sometimes, the xml received is very bad; e.g.:
        #   * unit price = 0, qty = 0, but price_subtotal = -200
        #   * unit price = 0, qty = 1, but price_subtotal = -200
        #   * unit price = 1, qty = 0, but price_subtotal = -200
        # for instance, when filling a down payment as an invoice line. The equation in the docstring is not
        # respected, and the result will not be correct, so we just follow the simple rule below:
        if net_price_unit is not None and price_subtotal != net_price_unit * (billed_qty / basis_qty) - allow_charge_amount:
            if net_price_unit == 0 and billed_qty == 0:
                quantity = 1
                price_unit = price_subtotal
            elif net_price_unit == 0:
                price_unit = price_subtotal / billed_qty
            elif billed_qty == 0:
                quantity = price_subtotal / price_unit

        return {
            'quantity': quantity,
            'price_unit': price_unit,
            'discount': discount,
            'product_uom_id': product_uom_id,
            'fixed_taxes_list': fixed_taxes_list,
        }

    def _import_retrieve_fixed_tax(self, invoice_line, fixed_tax_vals):
        """ Retrieve the fixed tax at import, iteratively search for a tax:
        1. not price_include matching the name and the amount
        2. not price_include matching the amount
        3. price_include matching the name and the amount
        4. price_include matching the amount
        """
        base_domain = [
            *self.env['account.journal']._check_company_domain(invoice_line.company_id),
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

    def _import_fill_invoice_line_taxes(self, tax_nodes, invoice_line, inv_line_vals, logs):
        # Taxes: all amounts are tax excluded, so first try to fetch price_include=False taxes,
        # if no results, try to fetch the price_include=True taxes. If results, need to adapt the price_unit.
        inv_line_vals['taxes'] = []
        for tax_node in tax_nodes:
            amount = float(tax_node.text)
            domain = [
                *self.env['account.journal']._check_company_domain(invoice_line.company_id),
                ('amount_type', '=', 'percent'),
                ('type_tax_use', '=', invoice_line.move_id.journal_id.type),
                ('amount', '=', amount),
            ]

            tax = False
            if hasattr(invoice_line, '_predict_specific_tax'):
                # company check is already done in the prediction query
                predicted_tax_id = invoice_line\
                    ._predict_specific_tax('percent', amount, invoice_line.move_id.journal_id.type)
                tax = self.env['account.tax'].browse(predicted_tax_id)
            if not tax:
                tax = self.env['account.tax'].search(domain + [('price_include', '=', False)], limit=1)
            if not tax:
                tax = self.env['account.tax'].search(domain + [('price_include', '=', True)], limit=1)

            if not tax:
                logs.append(_("Could not retrieve the tax: %s %% for line '%s'.", amount, invoice_line.name))
            else:
                inv_line_vals['taxes'].append(tax.id)
                if tax.price_include:
                    inv_line_vals['price_unit'] *= (1 + tax.amount / 100)

        # Handle Fixed Taxes
        for fixed_tax_vals in inv_line_vals['fixed_taxes_list']:
            tax = self._import_retrieve_fixed_tax(invoice_line, fixed_tax_vals)
            if not tax:
                # Nothing found: fix the price_unit s.t. line subtotal is matching the original invoice
                inv_line_vals['price_unit'] += fixed_tax_vals['tax_amount']
            elif tax.price_include:
                inv_line_vals['taxes'].append(tax.id)
                inv_line_vals['price_unit'] += tax.amount
            else:
                inv_line_vals['taxes'].append(tax.id)

        # Set the values on the line_form
        invoice_line.quantity = inv_line_vals['quantity']
        if not inv_line_vals.get('product_uom_id'):
            logs.append(
                _("Could not retrieve the unit of measure for line with label '%s'.", invoice_line.name))
        elif not invoice_line.product_id:
            # no product set on the line, no need to check uom compatibility
            invoice_line.product_uom_id = inv_line_vals['product_uom_id']
        elif inv_line_vals['product_uom_id'].category_id == invoice_line.product_id.product_tmpl_id.uom_id.category_id:
            # needed to check that the uom is compatible with the category of the product
            invoice_line.product_uom_id = inv_line_vals['product_uom_id']

        invoice_line.price_unit = inv_line_vals['price_unit']
        invoice_line.discount = inv_line_vals['discount']
        invoice_line.tax_ids = inv_line_vals['taxes'] or False
        return logs

    def _correct_invoice_tax_amount(self, tree, invoice):
        pass  # To be implemented by the format if needed

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
            invoice.message_post(body="ECOSIO: could not validate xml, formats only exist for invoice or credit notes")
            return
        if not ecosio_format:
            return
        response = soap_client.service.validate(xml_content, ecosio_format)

        report = []
        errors_cnt = 0
        for item in response['Result']:
            if item['artifactPath']:
                report.append(
                    Markup("<li><font style='color:Blue;'><strong>%s</strong></font></li>") % item['artifactPath'])
            for detail in item['Item']:
                if detail['errorLevel'] == 'WARN':
                    errors_cnt += 1
                    report.append(
                        Markup("<li><font style='color:Orange;'><strong>%s</strong></font></li>") % detail['errorText'])
                elif detail['errorLevel'] == 'ERROR':
                    errors_cnt += 1
                    report.append(
                        Markup("<li><font style='color:Tomato;'><strong>%s</strong></font></li>") % detail['errorText'])

        if errors_cnt == 0:
            invoice.message_post(body=Markup("<font style='color:Green;'><strong>ECOSIO: All clear for format %s!</strong></font>") % ecosio_format)
        else:
            invoice.message_post(
                body=Markup("<font style='color:Tomato;'><strong>ECOSIO ERRORS/WARNINGS for format %s</strong></font>: <ul>%s</<ul>") % (
                    ecosio_format,
                    Markup().join(report)
                )
            )
        return response

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
        return f"{invoice.name.replace('/', '_')}_factur_x.xml"

    def _export_invoice_ecosio_schematrons(self):
        return {
            'invoice': 'de.xrechnung:cii:2.2.0',
            'credit_note': 'de.xrechnung:cii:2.2.0',
        }

    def _export_invoice_constraints(self, invoice, vals):
        constraints = self._invoice_constraints_common(invoice)
        if invoice.move_type == 'out_invoice':
            # [BR-DE-1] An Invoice must contain information on "PAYMENT INSTRUCTIONS" (BG-16)
            # first check that a partner_bank_id exists, then check that there is an account number
            constraints.update({
                'seller_payment_instructions_1': self._check_required_fields(
                    vals['record'], 'partner_bank_id'
                ),
                'seller_payment_instructions_2': self._check_required_fields(
                    vals['record']['partner_bank_id'], 'sanitized_acc_number',
                    _("The field 'Sanitized Account Number' is required on the Recipient Bank.")
                ),
            })
        constraints.update({
            # [BR-08]-An Invoice shall contain the Seller postal address (BG-5).
            # [BR-09]-The Seller postal address (BG-5) shall contain a Seller country code (BT-40).
            'seller_postal_address': self._check_required_fields(
                vals['record']['company_id']['partner_id']['commercial_partner_id'], 'country_id'
            ),
            # [BR-CO-26]-In order for the buyer to automatically identify a supplier, the Seller identifier (BT-29),
            # the Seller legal registration identifier (BT-30) and/or the Seller VAT identifier (BT-31) shall be present.
            'seller_identifier': self._check_required_fields(
                vals['record']['company_id'], ['vat']  # 'siret'
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
                if vals['record']['partner_id']['country_id']['code'] == 'ES'
                    and vals['record']['partner_id']['zip']
                    and vals['record']['partner_id']['zip'][:2] in ['35', '38'] else None,
        })
        return constraints

    def _check_required_tax(self, vals):
        for line_vals in vals['invoice_line_vals_list']:
            line = line_vals['line']
            if not vals['tax_details']['tax_details_per_record'][line]['tax_details']:
                return _("You should include at least one tax per invoice line. [BR-CO-04]-Each Invoice line (BG-25) "
                         "shall be categorized with an Invoiced item VAT category code (BT-151).")

    def _check_non_0_rate_tax(self, vals):
        for line_vals in vals['tax_details']['tax_details_per_record']:
            tax_rate_list = line_vals.tax_ids.flatten_taxes_hierarchy().mapped("amount")
            if not any([rate > 0 for rate in tax_rate_list]):
                return _("When the Canary Island General Indirect Tax (IGIC) applies, the tax rate on "
                         "each invoice line should be greater than 0.")

    def _get_scheduled_delivery_time(self, invoice):
        # don't create a bridge only to get line.sale_line_ids.order_id.picking_ids.date_done
        # line.sale_line_ids.order_id.picking_ids.scheduled_date or line.sale_line_ids.order_id.commitment_date
        return invoice.delivery_date or invoice.invoice_date

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

        def grouping_key_generator(base_line, tax_values):
            tax = tax_values['tax_repartition_line'].tax_id
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
        tax_details = invoice._prepare_invoice_aggregated_taxes(grouping_key_generator=grouping_key_generator)

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

        buyer_siret = invoice.commercial_partner_id.company_registry
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
            'contract_reference': invoice.contract_reference if 'contract_reference' in invoice._fields and invoice.contract_reference else '',
            'document_context_id': "urn:cen.eu:en16931:2017#conformant#urn:factur-x.eu:1p0:extended",
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
            tax_detail_vals['calculated_amount'] = amount_currency if not invoice.currency_id.is_zero(amount_currency) else 0

            if tax_detail_vals.get('tax_category_code') == 'K':
                template_values['intracom_delivery'] = True
            # [BR - IC - 11] - In an Invoice with a VAT breakdown (BG-23) where the VAT category code (BT-118) is
            # "Intra-community supply" the Actual delivery date (BT-72) or the Invoicing period (BG-14) shall not be blank.
            if tax_detail_vals.get('tax_category_code') == 'K' and not template_values['scheduled_delivery_time']:
                date_range = self._get_invoicing_period(invoice)
                template_values['billing_start'] = min(date_range)
                template_values['billing_end'] = max(date_range)

        # Fixed taxes: add them as charges on the invoice lines
        for line_vals in template_values['invoice_line_vals_list']:
            line_vals['allowance_charge_vals_list'] = []
            for grouping_key, tax_detail in tax_details['tax_details_per_record'][line_vals['line']]['tax_details'].items():
                if grouping_key['amount_type'] == 'fixed':
                    line_vals['allowance_charge_vals_list'].append({
                        'indicator': 'true',
                        'reason': tax_detail['tax_name'],
                        'reason_code': 'AEO',
                        'amount': tax_detail['tax_amount_currency'],
                    })
            sum_fixed_taxes = sum(x['amount'] for x in line_vals['allowance_charge_vals_list'])
            line_vals['line_total_amount'] = line_vals['line'].price_subtotal + sum_fixed_taxes

            line_vals['quantity'] = line_vals['line'].quantity #/!\ The quantity is the line.quantity since we keep the unece_uom_code!

            # Invert the quantity and the gross_price_total_unit if a line has a negative price total
            if line_vals['line'].currency_id.compare_amounts(line_vals['gross_price_total_unit'], 0) == -1:
                line_vals['quantity'] *= -1
                line_vals['gross_price_total_unit'] *= -1
                line_vals['price_subtotal_unit'] *= -1

        # Fixed taxes: set the total adjusted amounts on the document level
        template_values['tax_basis_total_amount'] = tax_details['base_amount_currency']
        template_values['tax_total_amount'] = tax_details['tax_amount_currency']

        return template_values

    def _export_invoice(self, invoice):
        vals = self._export_invoice_vals(invoice.with_context(lang=invoice.partner_id.lang))
        errors = [constraint for constraint in self._export_invoice_constraints(invoice, vals).values() if constraint]
        xml_content = self.env['ir.qweb']._render('account_edi_ubl_cii.account_invoice_facturx_export_22', vals)
        return etree.tostring(cleanup_xml_node(xml_content), xml_declaration=True, encoding='UTF-8'), set(errors)

    # -------------------------------------------------------------------------
    # IMPORT
    # -------------------------------------------------------------------------

    def _import_fill_invoice_form(self, invoice, tree, qty_factor):
        logs = []

        if qty_factor == -1:
            logs.append(_("The invoice has been converted into a credit note and the quantities have been reverted."))

        # ==== partner_id ====

        role = invoice.journal_id.type == 'purchase' and 'SellerTradeParty' or 'BuyerTradeParty'
        name = self._find_value(f"//ram:{role}/ram:Name", tree)
        mail = self._find_value(f"//ram:{role}//ram:URIID", tree)
        vat = self._find_value(f"//ram:{role}/ram:SpecifiedTaxRegistration/ram:ID[string-length(text()) > 5]", tree)
        phone = self._find_value(f"//ram:{role}/ram:DefinedTradeContact/ram:TelephoneUniversalCommunication/ram:CompleteNumber", tree)
        country_code = self._find_value(f'//ram:{role}/ram:PostalTradeAddress//ram:CountryID', tree)
        self._import_retrieve_and_fill_partner(invoice, name=name, phone=phone, mail=mail, vat=vat, country_code=country_code)

        # ==== currency_id ====

        currency_code_node = tree.find('.//{*}InvoiceCurrencyCode')
        if currency_code_node is not None:
            currency = self.env['res.currency'].with_context(active_test=False).search([
                ('name', '=', currency_code_node.text),
            ], limit=1)
            if currency:
                if not currency.active:
                    logs.append(_("The currency '%s' is not active.", currency.name))
                invoice.currency_id = currency
            else:
                logs.append(_("Could not retrieve currency: %s. Did you enable the multicurrency option and "
                              "activate the currency?", currency_code_node.text))

        # ==== Bank Details ====

        bank_detail_nodes = tree.findall('.//{*}SpecifiedTradeSettlementPaymentMeans')
        bank_details = [
            bank_detail_node.findtext('{*}PayeePartyCreditorFinancialAccount/{*}IBANID')
            or bank_detail_node.findtext('{*}PayeePartyCreditorFinancialAccount/{*}ProprietaryID')
            for bank_detail_node in bank_detail_nodes
        ]

        if bank_details:
            self._import_retrieve_and_fill_partner_bank_details(invoice, bank_details=bank_details)

        # ==== Reference ====

        ref_node = tree.find('./{*}ExchangedDocument/{*}ID')
        if ref_node is not None:
            invoice.ref = ref_node.text

        # ==== Invoice origin ====

        invoice_origin_node = tree.find('.//{*}BuyerOrderReferencedDocument/{*}IssuerAssignedID')
        if invoice_origin_node is not None:
            invoice.invoice_origin = invoice_origin_node.text

        # === Note/narration ====

        narration = ""
        note_node = tree.find('./{*}ExchangedDocument/{*}IncludedNote/{*}Content')
        if note_node is not None and note_node.text:
            narration += note_node.text + "\n"

        payment_terms_node = tree.find('.//{*}SpecifiedTradePaymentTerms/{*}Description')
        if payment_terms_node is not None and payment_terms_node.text:
            narration += payment_terms_node.text + "\n"

        invoice.narration = narration

        # ==== payment_reference ====

        payment_reference_node = tree.find('./{*}SupplyChainTradeTransaction/{*}ApplicableHeaderTradeSettlement/{*}PaymentReference')
        if payment_reference_node is not None:
            invoice.payment_reference = payment_reference_node.text

        # ==== invoice_date ====

        invoice_date_node = tree.find('./{*}ExchangedDocument/{*}IssueDateTime/{*}DateTimeString')
        if invoice_date_node is not None and invoice_date_node.text:
            date_str = invoice_date_node.text.strip()
            date_obj = datetime.strptime(date_str, DEFAULT_FACTURX_DATE_FORMAT)
            invoice.invoice_date = date_obj.strftime(DEFAULT_SERVER_DATE_FORMAT)

        # ==== invoice_date_due ====

        invoice_date_due_node = tree.find('.//{*}SpecifiedTradePaymentTerms/{*}DueDateDateTime/{*}DateTimeString')
        if invoice_date_due_node is not None and invoice_date_due_node.text:
            date_str = invoice_date_due_node.text.strip()
            date_obj = datetime.strptime(date_str, DEFAULT_FACTURX_DATE_FORMAT)
            invoice.invoice_date_due = date_obj.strftime(DEFAULT_SERVER_DATE_FORMAT)

        # ==== invoice_line_ids: AllowanceCharge (document level) ====

        logs += self._import_fill_invoice_allowance_charge(tree, invoice, qty_factor)

        # ==== Prepaid amount ====

        prepaid_node = tree.find('.//{*}ApplicableHeaderTradeSettlement/'
                                 '{*}SpecifiedTradeSettlementHeaderMonetarySummation/{*}TotalPrepaidAmount')
        logs += self._import_log_prepaid_amount(invoice, prepaid_node, qty_factor)

        # ==== invoice_line_ids ====

        line_nodes = tree.findall('./{*}SupplyChainTradeTransaction/{*}IncludedSupplyChainTradeLineItem')
        if line_nodes is not None:
            for invl_el in line_nodes:
                invoice_line = invoice.invoice_line_ids.create({'move_id': invoice.id})
                invl_logs = self._import_fill_invoice_line_form(invoice.journal_id, invl_el, invoice, invoice_line, qty_factor)
                logs += invl_logs

        return logs

    def _import_fill_invoice_line_form(self, journal, tree, invoice_form, invoice_line, qty_factor):
        logs = []

        # Product.
        name = self._find_value('.//ram:SpecifiedTradeProduct/ram:Name', tree)
        invoice_line.product_id = self.env['product.product']._retrieve_product(
            default_code=self._find_value('.//ram:SpecifiedTradeProduct/ram:SellerAssignedID', tree),
            name=name,
            barcode=self._find_value('.//ram:SpecifiedTradeProduct/ram:GlobalID', tree)
        )
        # force original line description instead of the one copied from product's Sales Description
        if name:
            invoice_line.name = name

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
        inv_line_vals = self._import_fill_invoice_line_values(tree, xpath_dict, invoice_line, qty_factor)
        # retrieve tax nodes
        tax_nodes = tree.findall('.//{*}ApplicableTradeTax/{*}RateApplicablePercent')
        return self._import_fill_invoice_line_taxes(tax_nodes, invoice_line, inv_line_vals, logs)

    # -------------------------------------------------------------------------
    # IMPORT : helpers
    # -------------------------------------------------------------------------

    def _get_import_document_amount_sign(self, tree):
        """
        In factur-x, an invoice has code 380 and a credit note has code 381. However, a credit note can be expressed
        as an invoice with negative amounts. For this case, we need a factor to take the opposite of each quantity
        in the invoice.
        """
        move_type_code = tree.find('.//{*}ExchangedDocument/{*}TypeCode')
        if move_type_code is None:
            return None, None
        if move_type_code.text == '381':
            return 'refund', 1
        if move_type_code.text == '380':
            amount_node = tree.find('.//{*}SpecifiedTradeSettlementHeaderMonetarySummation/{*}TaxBasisTotalAmount')
            if amount_node is not None and float(amount_node.text) < 0:
                return 'refund', -1
            return 'invoice', 1

```

## File: models\account_edi_xml_ubl_20.py

```python
# -*- coding: utf-8 -*-
from collections import defaultdict
from lxml import etree

from odoo import models, _
from odoo.tools import html2plaintext, cleanup_xml_node

UBL_NAMESPACES = {
    'cbc': "urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2",
    'cac': "urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2",
}


class AccountEdiXmlUBL20(models.AbstractModel):
    _name = "account.edi.xml.ubl_20"
    _inherit = 'account.edi.common'
    _description = "UBL 2.0"

    def _find_value(self, xpath, tree, nsmap=False):
        # EXTENDS account.edi.common
        return super()._find_value(xpath, tree, UBL_NAMESPACES)

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
        # [BR-CO-09] if the PartyTaxScheme/TaxScheme/ID == 'VAT', CompanyID must start with a country code prefix.
        # In some countries however, the CompanyID can be with or without country code prefix and still be perfectly
        # valid (RO, HU, non-EU countries).
        # We have to handle their cases by changing the TaxScheme/ID to 'something other than VAT',
        # preventing the trigger of the rule.
        tax_scheme_id = 'VAT'
        if (
            partner.country_id
            and partner.vat and not partner.vat[:2].isalpha()
        ):
            tax_scheme_id = 'NOT_EU_VAT'
        return [{
            'registration_name': partner.name,
            'company_id': partner.vat,
            'registration_address_vals': self._get_partner_address_vals(partner),
            'tax_scheme_vals': {'id': tax_scheme_id},
        }]

    def _get_partner_party_legal_entity_vals_list(self, partner):
        return [{
            'commercial_partner': partner,
            'registration_name': partner.name,
            'company_id': partner.vat,
            'registration_address_vals': self._get_partner_address_vals(partner),
        }]

    def _get_partner_contact_vals(self, partner):
        return {
            'id': partner.id,
            'name': partner.name,
            'telephone': partner.phone or partner.mobile,
            'electronic_mail': partner.email,
        }

    def _get_partner_person_vals(self, partner):
        """
        This is optional and meant to be overridden when required under the form:
        {
            'first_name': str,
            'family_name': str,
        }.
        Should return a dict.
        """
        return {}

    def _get_partner_party_vals(self, partner, role):
        return {
            'partner': partner,
            'party_identification_vals': self._get_partner_party_identification_vals_list(partner.commercial_partner_id),
            'party_name_vals': [{'name': partner.display_name}],
            'postal_address_vals': self._get_partner_address_vals(partner),
            'party_tax_scheme_vals': self._get_partner_party_tax_scheme_vals_list(partner.commercial_partner_id, role),
            'party_legal_entity_vals': self._get_partner_party_legal_entity_vals_list(partner.commercial_partner_id),
            'contact_vals': self._get_partner_contact_vals(partner),
            'person_vals': self._get_partner_person_vals(partner),
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

    def _get_additional_document_reference_list(self, invoice):
        """
        This is optional and meant to be overridden when required under the form:
        {
            'id': str,
            'issue_date': str,
            'document_type_code': str,
            'document_type': str,
            'document_description': str,
        }.
        Should return a list.
        """
        return []

    def _get_delivery_vals_list(self, invoice):
        # the data is optional, except for ubl bis3 (see the override, where we need to set a default delivery address)
        return [{
            'actual_delivery_date': invoice.delivery_date,
            'delivery_location_vals': {
                'delivery_address_vals': self._get_partner_address_vals(invoice.partner_shipping_id),
            },
            'delivery_party_vals': self._get_partner_party_vals(invoice.partner_shipping_id, 'delivery') if invoice.partner_shipping_id else {},
        }]

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
        payment_means_code, payment_means_name = (30, 'credit transfer') if invoice.move_type == 'out_invoice' else (57, 'standing agreement')
        # in Denmark payment code 30 is not allowed. we hardcode it to 1 ("unknown") for now
        # as we cannot deduce this information from the invoice
        if invoice.partner_id.country_code == 'DK':
            payment_means_code, payment_means_name = 1, 'unknown'

        vals = {
            'payment_means_code': payment_means_code,
            'payment_means_code_attrs': {'name': payment_means_name},
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
            # The payment term's note is automatically embedded in a <p> tag in Odoo
            return [{'note_vals': [{'note': html2plaintext(payment_term.note)}]}]
        else:
            return []

    def _get_invoice_tax_totals_vals_list(self, invoice, taxes_vals):
        tax_totals_vals = {
            'currency': invoice.currency_id,
            'currency_dp': self._get_currency_decimal_places(invoice.currency_id),
            'tax_amount': taxes_vals['tax_amount_currency'],
            'tax_subtotal_vals': [],
        }

        # If it's not on the whole invoice, don't manage the EPD.
        epd_tax_to_discount = {}
        if not taxes_vals.get('invoice_line'):
            epd_tax_to_discount = self._get_early_payment_discount_grouped_by_tax_rate(invoice)
            epd_base_tax_amounts = defaultdict(lambda: {
                'base_amount_currency': 0.0,
                'tax_amount_currency': 0.0,
            })
            if epd_tax_to_discount:
                for percentage, base_amount_currency in epd_tax_to_discount.items():
                    epd_base_tax_amounts[percentage]['base_amount_currency'] += base_amount_currency
                epd_accounted_tax_amount = 0.0
                for percentage, amounts in epd_base_tax_amounts.items():
                    amounts['tax_amount_currency'] = invoice.currency_id.round(
                        amounts['base_amount_currency'] * percentage / 100.0)
                    epd_accounted_tax_amount += amounts['tax_amount_currency']

        for grouping_key, vals in taxes_vals['tax_details'].items():
            if grouping_key['tax_amount_type'] != 'fixed':
                subtotal = {
                    'currency': invoice.currency_id,
                    'currency_dp': self._get_currency_decimal_places(invoice.currency_id),
                    'taxable_amount': vals['base_amount_currency'],
                    'tax_amount': vals['tax_amount_currency'],
                    'percent': vals['tax_category_percent'],
                    'tax_category_vals': vals['_tax_category_vals_'],
                }
                if epd_tax_to_discount:
                    # early payment discounts: need to recompute the tax/taxable amounts
                    epd_base_amount = epd_base_tax_amounts.get(subtotal['percent'], {}).get('base_amount_currency', 0.0)
                    taxable_amount_after_epd = subtotal['taxable_amount'] - epd_base_amount
                    subtotal.update({
                        'taxable_amount': taxable_amount_after_epd,
                    })
                tax_totals_vals['tax_subtotal_vals'].append(subtotal)

        if epd_tax_to_discount:
            # early payment discounts: hence, need to add a subtotal section
            tax_totals_vals['tax_subtotal_vals'].append({
                'currency': invoice.currency_id,
                'currency_dp': invoice.currency_id.decimal_places,
                'taxable_amount': sum(epd_tax_to_discount.values()),
                'tax_amount': 0.0,
                'tax_category_vals': {
                    'id': 'E',
                    'percent': 0.0,
                    'tax_scheme_vals': {
                        'id': "VAT",
                    },
                    'tax_exemption_reason': "Exempt from tax",
                },
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
            'description': description,
            'name': product.name or description,
            'sellers_item_identification_vals': {'id': product.code},
            'classified_tax_category_vals': tax_category_vals_list,
        }

    def _get_document_allowance_charge_vals_list(self, invoice):
        """
        https://docs.peppol.eu/poacc/billing/3.0/bis/#_document_level_allowance_or_charge
        Usage for early payment discounts:
        * Add one document level Allowance per tax rate (VAT included)
        * Add one document level Charge (VAT excluded) with amount = the total sum of the early payment discount
        The difference between these is the cash discount in case of early payment.
        """
        vals_list = []
        # Early Payment Discount
        epd_tax_to_discount = self._get_early_payment_discount_grouped_by_tax_rate(invoice)
        if epd_tax_to_discount:
            # One Allowance per tax rate (VAT included)
            for tax_amount, discount_amount in epd_tax_to_discount.items():
                vals_list.append({
                    'charge_indicator': 'false',
                    'allowance_charge_reason_code': '66',
                    'allowance_charge_reason': _("Conditional cash/payment discount"),
                    'amount': discount_amount,
                    'currency_dp': 2,
                    'currency_name': invoice.currency_id.name,
                    'tax_category_vals': [{
                        'id': 'S',
                        'percent': tax_amount,
                        'tax_scheme_vals': {'id': 'VAT'},
                    }],
                })
            # One global Charge (VAT exempted)
            vals_list.append({
                'charge_indicator': 'true',
                'allowance_charge_reason_code': 'ZZZ',
                'allowance_charge_reason': _("Conditional cash/payment discount"),
                'amount': sum(epd_tax_to_discount.values()),
                'currency_dp': 2,
                'currency_name': invoice.currency_id.name,
                'tax_category_vals': [{
                    'id': 'E',
                    'percent': 0.0,
                    'tax_scheme_vals': {'id': 'VAT'},
                }],
            })
        return vals_list

    def _get_invoice_line_allowance_vals_list(self, line, tax_values_list=None):
        """ Method used to fill the cac:{Invoice,CreditNote,DebitNote}Line>cac:AllowanceCharge node.

        Allowances are distinguished from charges using the ChargeIndicator node with 'false' as value.

        Note that allowance charges do not exist for credit notes in UBL 2.0, so if we apply discount in Odoo
        the net price will not be consistent with the unit price, but we cannot do anything about it

        :param line:    An invoice line.
        :return:        A list of python dictionaries.
        """
        fixed_tax_charge_vals_list = []
        for grouping_key, tax_details in tax_values_list['tax_details'].items():
            if grouping_key['tax_amount_type'] == 'fixed':
                fixed_tax_charge_vals_list.append({
                    'currency_name': line.currency_id.name,
                    'currency_dp': self._get_currency_decimal_places(line.currency_id),
                    'charge_indicator': 'true',
                    'allowance_charge_reason_code': 'AEO',
                    'allowance_charge_reason': tax_details['tax_name'],
                    'amount': tax_details['tax_amount_currency'],
                })

        if not line.discount:
            return fixed_tax_charge_vals_list

        # Price subtotal with discount subtracted:
        net_price_subtotal = line.price_subtotal
        # Price subtotal without discount subtracted:
        if line.discount == 100.0:
            gross_price_subtotal = 0.0
        else:
            gross_price_subtotal = line.currency_id.round(net_price_subtotal / (1.0 - (line.discount or 0.0) / 100.0))

        allowance_vals = {
            'currency_name': line.currency_id.name,
            'currency_dp': self._get_currency_decimal_places(line.currency_id),

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
            'currency_dp': self._get_currency_decimal_places(line.currency_id),

            # The price of an item, exclusive of VAT, after subtracting item price discount.
            'price_amount': round(gross_price_unit, 10),
            'product_price_dp': self.env['decimal.precision'].precision_get('Product Price'),

            # The number of item units to which the price applies.
            # setting to None -> the xml will not comprise the BaseQuantity (it's not mandatory)
            'base_quantity': None,
            'base_quantity_attrs': {'unitCode': uom},
        }

    def _get_invoice_line_tax_totals_vals_list(self, line, taxes_vals):
        """ Method used to fill the cac:TaxTotal node on a line level.
        Uses the same method as the invoice TaxTotal, but can be overridden in other formats.
        """
        return self._get_invoice_tax_totals_vals_list(line.move_id, taxes_vals)

    def _get_invoice_line_vals(self, line, line_id, taxes_vals):
        """ Method used to fill the cac:{Invoice,CreditNote,DebitNote}Line node.
        It provides information about the document line.

        :param line:    A document line.
        :return:        A python dictionary.
        """
        allowance_charge_vals_list = self._get_invoice_line_allowance_vals_list(line, tax_values_list=taxes_vals)

        uom = super()._get_uom_unece_code(line)
        total_fixed_tax_amount = sum(
            vals['amount']
            for vals in allowance_charge_vals_list
            if vals.get('charge_indicator') == 'true'
        )
        period_vals = {}
        # deferred_start_date & deferred_end_date are enterprise-only fields
        if line._fields.get('deferred_start_date') and (line.deferred_start_date or line.deferred_end_date):
            period_vals.update({'start_date': line.deferred_start_date})
            period_vals.update({'end_date': line.deferred_end_date})
        return {
            'currency': line.currency_id,
            'currency_dp': self._get_currency_decimal_places(line.currency_id),
            'id': line_id + 1,
            'line_quantity': line.quantity,
            'line_quantity_attrs': {'unitCode': uom},
            'line_extension_amount': line.price_subtotal + total_fixed_tax_amount,
            'allowance_charge_vals': allowance_charge_vals_list,
            'tax_total_vals': self._get_invoice_line_tax_totals_vals_list(line, taxes_vals),
            'item_vals': self._get_invoice_line_item_vals(line, taxes_vals),
            'price_vals': self._get_invoice_line_price_vals(line),
            'invoice_period_vals_list': [period_vals] if period_vals else []
        }

    def _get_invoice_monetary_total_vals(self, invoice, taxes_vals, line_extension_amount, allowance_total_amount, charge_total_amount):
        """ Method used to fill the cac:{Legal,Requested}MonetaryTotal node"""
        return {
            'currency': invoice.currency_id,
            'currency_dp': self._get_currency_decimal_places(invoice.currency_id),
            'line_extension_amount': line_extension_amount,
            'tax_exclusive_amount': taxes_vals['base_amount_currency'],
            'tax_inclusive_amount': invoice.amount_total,
            'allowance_total_amount': allowance_total_amount or None,
            'charge_total_amount': charge_total_amount or None,
            'prepaid_amount': invoice.amount_total - invoice.amount_residual,
            'payable_amount': invoice.amount_residual,
        }

    def _apply_invoice_tax_filter(self, base_line, tax_values):
        """
            To be overridden to apply a specific tax filter
        """
        return True

    def _apply_invoice_line_filter(self, invoice_line):
        """
            To be overridden to apply a specific invoice line filter
        """
        return True

    def _get_early_payment_discount_grouped_by_tax_rate(self, invoice):
        """
        Get the early payment discounts grouped by the tax rate of the product it is linked to
        :returns {float: float}: mapping tax amounts to early payment discount amounts
        """
        if invoice.invoice_payment_term_id.early_pay_discount_computation != 'mixed':
            return {}
        tax_to_discount = defaultdict(lambda: 0)
        for line in invoice.line_ids.filtered(lambda l: l.display_type == 'epd'):
            for tax in line.tax_ids:
                tax_to_discount[tax.amount] += line.amount_currency
        return tax_to_discount

    def _export_invoice_vals(self, invoice):
        def grouping_key_generator(base_line, tax_values):
            tax = tax_values['tax_repartition_line'].tax_id
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
        taxes_vals = invoice._prepare_invoice_aggregated_taxes(
            grouping_key_generator=grouping_key_generator,
            filter_tax_values_to_apply=self._apply_invoice_tax_filter,
            filter_invl_to_apply=self._apply_invoice_line_filter,
        )

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

        invoice_lines = invoice.invoice_line_ids.filtered(lambda line: line.display_type not in ('line_note', 'line_section') and line._check_edi_line_tax_required())
        document_allowance_charge_vals_list = self._get_document_allowance_charge_vals_list(invoice)
        invoice_line_vals_list = []
        for line_id, line in enumerate(invoice_lines):
            line_taxes_vals = taxes_vals['tax_details_per_record'][line]
            line_vals = self._get_invoice_line_vals(line, line_id, {**line_taxes_vals, 'invoice_line': line})
            invoice_line_vals_list.append(line_vals)

            line_extension_amount += line_vals['line_extension_amount']

        # Compute the total allowance/charge amounts.
        allowance_total_amount = 0.0
        charge_total_amount = 0.0
        for allowance_charge_vals in document_allowance_charge_vals_list:
            if allowance_charge_vals['charge_indicator'] == 'false':
                allowance_total_amount += allowance_charge_vals['amount']
            else:
                charge_total_amount += allowance_charge_vals['amount']

        supplier = invoice.company_id.partner_id.commercial_partner_id
        customer = invoice.partner_id

        # OrderReference/SalesOrderID (sales_order_id) is optional
        sales_order_id = 'sale_line_ids' in invoice.invoice_line_ids._fields \
                         and ",".join(invoice.invoice_line_ids.sale_line_ids.order_id.mapped('name'))
        # OrderReference/ID (order_reference) is mandatory inside the OrderReference node !
        order_reference = invoice.ref or invoice.name

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
            'PaymentTermsType_template': 'account_edi_ubl_cii.ubl_20_PaymentTermsType',
            'TaxCategoryType_template': 'account_edi_ubl_cii.ubl_20_TaxCategoryType',
            'TaxTotalType_template': 'account_edi_ubl_cii.ubl_20_TaxTotalType',
            'AllowanceChargeType_template': 'account_edi_ubl_cii.ubl_20_AllowanceChargeType',
            'SignatureType_template': 'account_edi_ubl_cii.ubl_20_SignatureType',
            'ResponseType_template': 'account_edi_ubl_cii.ubl_20_ResponseType',
            'DeliveryType_template': 'account_edi_ubl_cii.ubl_20_DeliveryType',
            'MonetaryTotalType_template': 'account_edi_ubl_cii.ubl_20_MonetaryTotalType',
            'InvoiceLineType_template': 'account_edi_ubl_cii.ubl_20_InvoiceLineType',
            'CreditNoteLineType_template': 'account_edi_ubl_cii.ubl_20_CreditNoteLineType',
            'DebitNoteLineType_template': 'account_edi_ubl_cii.ubl_20_DebitNoteLineType',
            'InvoiceType_template': 'account_edi_ubl_cii.ubl_20_InvoiceType',
            'CreditNoteType_template': 'account_edi_ubl_cii.ubl_20_CreditNoteType',
            'DebitNoteType_template': 'account_edi_ubl_cii.ubl_20_DebitNoteType',

            'vals': {
                'ubl_version_id': 2.0,
                'id': invoice.name,
                'issue_date': invoice.invoice_date,
                'due_date': invoice.invoice_date_due,
                'note_vals': self._get_note_vals_list(invoice),
                'order_reference': order_reference,
                'sales_order_id': sales_order_id,
                'accounting_supplier_party_vals': {
                    'party_vals': self._get_partner_party_vals(supplier, role='supplier'),
                },
                'accounting_customer_party_vals': {
                    'party_vals': self._get_partner_party_vals(customer, role='customer'),
                },
                'invoice_period_vals_list': self._get_invoice_period_vals_list(invoice),
                'additional_document_reference_list': self._get_additional_document_reference_list(invoice),
                'delivery_vals_list': self._get_delivery_vals_list(invoice),
                'payment_means_vals_list': self._get_invoice_payment_means_vals_list(invoice),
                'payment_terms_vals': self._get_invoice_payment_terms_vals_list(invoice),
                # allowances at the document level, the allowances on invoices (eg. discount) are on line_vals
                'allowance_charge_vals': document_allowance_charge_vals_list,
                'tax_total_vals': self._get_invoice_tax_totals_vals_list(invoice, taxes_vals),
                'monetary_total_vals': self._get_invoice_monetary_total_vals(
                    invoice,
                    taxes_vals,
                    line_extension_amount,
                    allowance_total_amount,
                    charge_total_amount,
                ),
                'line_vals': invoice_line_vals_list,
                'currency_dp': self._get_currency_decimal_places(invoice.currency_id),  # currency decimal places
            },
        }

        # Document type specific settings
        if 'debit_origin_id' in self.env['account.move']._fields and invoice.debit_origin_id:
            vals['document_type'] = 'debit_note'
            vals['main_template'] = 'account_edi_ubl_cii.ubl_20_DebitNote'
            vals['vals']['document_type_code'] = 383
        elif invoice.move_type == 'out_refund':
            vals['document_type'] = 'credit_note'
            vals['main_template'] = 'account_edi_ubl_cii.ubl_20_CreditNote'
            vals['vals']['document_type_code'] = 381
        else: # invoice.move_type == 'out_invoice'
            vals['document_type'] = 'invoice'
            vals['main_template'] = 'account_edi_ubl_cii.ubl_20_Invoice'
            vals['vals']['document_type_code'] = 380

        return vals

    def _get_note_vals_list(self, invoice):
        return [{'note': html2plaintext(invoice.narration)}] if invoice.narration else []

    def _export_invoice_constraints(self, invoice, vals):
        constraints = self._invoice_constraints_common(invoice)
        constraints.update({
            'ubl20_supplier_name_required': self._check_required_fields(vals['supplier'], 'name'),
            'ubl20_customer_name_required': self._check_required_fields(vals['customer'].commercial_partner_id, 'name'),
            'ubl20_invoice_name_required': self._check_required_fields(invoice, 'name'),
            'ubl20_invoice_date_required': self._check_required_fields(invoice, 'invoice_date'),
        })
        return constraints

    def _export_invoice(self, invoice):
        vals = self._export_invoice_vals(invoice.with_context(lang=invoice.partner_id.lang))
        errors = [constraint for constraint in self._export_invoice_constraints(invoice, vals).values() if constraint]
        xml_content = self.env['ir.qweb']._render(vals['main_template'], vals)
        return etree.tostring(cleanup_xml_node(xml_content), xml_declaration=True, encoding='UTF-8'), set(errors)

    def _get_document_type_code_vals(self, invoice, invoice_data):
        """Returns the values used for the `DocumentTypeCode` node"""
        # To be overriden by custom format if required
        return {'attrs': {}, 'value': None}

    # -------------------------------------------------------------------------
    # IMPORT
    # -------------------------------------------------------------------------

    def _import_retrieve_partner_vals(self, tree, role):
        """ Returns a dict of values that will be used to retrieve the partner """
        return {
            'vat': self._find_value(f'.//cac:Accounting{role}Party/cac:Party//cbc:CompanyID[string-length(text()) > 5]', tree),
            'phone': self._find_value(f'.//cac:Accounting{role}Party/cac:Party//cbc:Telephone', tree),
            'mail': self._find_value(f'.//cac:Accounting{role}Party/cac:Party//cbc:ElectronicMail', tree),
            'name': self._find_value(f'.//cac:Accounting{role}Party/cac:Party//cbc:Name', tree) or
                    self._find_value(f'.//cac:Accounting{role}Party/cac:Party//cbc:RegistrationName', tree),
            'country_code': self._find_value(f'.//cac:Accounting{role}Party/cac:Party//cac:Country//cbc:IdentificationCode', tree),
            'street': self._find_value(f'.//cac:Accounting{role}Party/cac:Party//cbc:StreetName', tree),
            'street2': self._find_value(f'.//cac:Accounting{role}Party/cac:Party//cbc:AdditionalStreetName', tree),
            'city': self._find_value(f'.//cac:Accounting{role}Party/cac:Party//cbc:CityName', tree),
            'zip_code': self._find_value(f'.//cac:Accounting{role}Party/cac:Party//cbc:PostalZone', tree),
        }

    def _import_fill_invoice_form(self, invoice, tree, qty_factor):
        logs = []

        if qty_factor == -1:
            logs.append(_("The invoice has been converted into a credit note and the quantities have been reverted."))

        # ==== partner_id ====

        role = "Customer" if invoice.journal_id.type == 'sale' else "Supplier"
        partner_vals = self._import_retrieve_partner_vals(tree, role)
        self._import_retrieve_and_fill_partner(invoice, **partner_vals)

        # ==== currency_id ====

        currency_code_node = tree.find('.//{*}DocumentCurrencyCode')
        if currency_code_node is not None:
            currency = self.env['res.currency'].with_context(active_test=False).search([
                ('name', '=', currency_code_node.text),
            ], limit=1)
            if currency:
                if not currency.active:
                    logs.append(_("The currency '%s' is not active.", currency.name))
                invoice.currency_id = currency
            else:
                logs.append(_("Could not retrieve currency: %s. Did you enable the multicurrency option "
                              "and activate the currency?", currency_code_node.text))

        # ==== invoice_date ====

        invoice_date_node = tree.find('./{*}IssueDate')
        if invoice_date_node is not None and invoice_date_node.text:
            invoice.invoice_date = invoice_date_node.text

        # ==== invoice_date_due ====

        for xpath in ('./{*}DueDate', './/{*}PaymentDueDate'):
            invoice_date_due_node = tree.find(xpath)
            if invoice_date_due_node is not None and invoice_date_due_node.text:
                invoice.invoice_date_due = invoice_date_due_node.text
                break

        # ==== Bank Details ====

        bank_detail_nodes = tree.findall('.//{*}PaymentMeans')
        bank_details = [bank_detail_node.findtext('{*}PayeeFinancialAccount/{*}ID') for bank_detail_node in bank_detail_nodes]

        if bank_details:
            self._import_retrieve_and_fill_partner_bank_details(invoice, bank_details=bank_details)

        # ==== Delivery ====
        delivery_date = tree.find('.//{*}Delivery/{*}ActualDeliveryDate')
        invoice.delivery_date = delivery_date is not None and delivery_date.text

        # ==== Reference ====

        ref_node = tree.find('./{*}ID')
        if ref_node is not None:
            if invoice.is_sale_document(include_receipts=True) and invoice.quick_edit_mode:
                invoice.name = ref_node.text
            else:
                invoice.ref = ref_node.text

        # ==== Invoice origin ====

        invoice_origin_node = tree.find('./{*}OrderReference/{*}ID')
        if invoice_origin_node is not None:
            invoice.invoice_origin = invoice_origin_node.text

        # === Note/narration ====

        narration = ""
        note_node = tree.find('./{*}Note')
        if note_node is not None and note_node.text:
            narration += f"<p>{note_node.text}</p>"

        payment_terms_node = tree.find('./{*}PaymentTerms/{*}Note')  # e.g. 'Payment within 10 days, 2% discount'
        if payment_terms_node is not None and payment_terms_node.text:
            narration += f"<p>{payment_terms_node.text}</p>"

        invoice.narration = narration

        # ==== payment_reference ====

        payment_reference_node = tree.find('./{*}PaymentMeans/{*}PaymentID')
        if payment_reference_node is not None:
            invoice.payment_reference = payment_reference_node.text

        # ==== invoice_incoterm_id ====

        incoterm_code_node = tree.find('./{*}TransportExecutionTerms/{*}DeliveryTerms/{*}ID')
        if incoterm_code_node is not None:
            incoterm = self.env['account.incoterms'].search([('code', '=', incoterm_code_node.text)], limit=1)
            if incoterm:
                invoice.invoice_incoterm_id = incoterm

        # ==== invoice_line_ids: AllowanceCharge (document level) ====

        logs += self._import_fill_invoice_allowance_charge(tree, invoice, qty_factor)

        # ==== Prepaid amount ====

        prepaid_node = tree.find('./{*}LegalMonetaryTotal/{*}PrepaidAmount')
        logs += self._import_log_prepaid_amount(invoice, prepaid_node, qty_factor)

        # ==== invoice_line_ids: InvoiceLine/CreditNoteLine ====

        invoice_line_tag = 'InvoiceLine' if invoice.move_type in ('in_invoice', 'out_invoice') or qty_factor == -1 else 'CreditNoteLine'
        for i, invl_el in enumerate(tree.findall('./{*}' + invoice_line_tag)):
            invoice_line = invoice.invoice_line_ids.create({'move_id': invoice.id})
            invl_logs = self._import_fill_invoice_line_form(invl_el, invoice_line, qty_factor)
            logs += invl_logs

        return logs

    def _import_fill_invoice_line_form(self, tree, invoice_line, qty_factor):
        logs = []

        # Product.
        invoice_line.product_id = self.env['product.product']._retrieve_product(
            default_code=self._find_value('./cac:Item/cac:SellersItemIdentification/cbc:ID', tree),
            name=self._find_value('./cac:Item/cbc:Name', tree),
            barcode=self._find_value("./cac:Item/cac:StandardItemIdentification/cbc:ID[@schemeID='0160']", tree),
            company=invoice_line.company_id
        )
        # Description
        description_node = tree.find('./{*}Item/{*}Description')
        name_node = tree.find('./{*}Item/{*}Name')
        if description_node is not None:
            invoice_line.name = description_node.text
        elif name_node is not None:
            invoice_line.name = name_node.text  # Fallback on Name if Description is not found.

        # Start and End date (enterprise fields)
        if invoice_line._fields.get('deferred_start_date'):
            start_date = tree.find('./{*}InvoicePeriod/{*}StartDate')
            end_date = tree.find('./{*}InvoicePeriod/{*}EndDate')
            if start_date is not None and end_date is not None:  # there is a constraint forcing none or the two to be set
                invoice_line.write({
                    'deferred_start_date': start_date.text,
                    'deferred_end_date': end_date.text,
                })
        xpath_dict = {
            'basis_qty': [
                './{*}Price/{*}BaseQuantity',
            ],
            'gross_price_unit': './{*}Price/{*}AllowanceCharge/{*}BaseAmount',
            'rebate': './{*}Price/{*}AllowanceCharge/{*}Amount',
            'net_price_unit': './{*}Price/{*}PriceAmount',
            'billed_qty':  './{*}InvoicedQuantity' if invoice_line.move_id.move_type in ('in_invoice', 'out_invoice') or qty_factor == -1 else './{*}CreditedQuantity',
            'allowance_charge': './/{*}AllowanceCharge',
            'allowance_charge_indicator': './{*}ChargeIndicator',
            'allowance_charge_amount': './{*}Amount',
            'allowance_charge_reason': './{*}AllowanceChargeReason',
            'allowance_charge_reason_code': './{*}AllowanceChargeReasonCode',
            'line_total_amount': './{*}LineExtensionAmount',
        }

        # Taxes
        inv_line_vals = self._import_fill_invoice_line_values(tree, xpath_dict, invoice_line, qty_factor)
        # retrieve tax nodes
        tax_nodes = tree.findall('.//{*}Item/{*}ClassifiedTaxCategory/{*}Percent')
        if not tax_nodes:
            for elem in tree.findall('.//{*}TaxTotal'):
                percentage_nodes = elem.findall('.//{*}TaxSubtotal/{*}TaxCategory/{*}Percent')
                if not percentage_nodes:
                    percentage_nodes = elem.findall('.//{*}TaxSubtotal/{*}Percent')
                tax_nodes += percentage_nodes
        return self._import_fill_invoice_line_taxes(tax_nodes, invoice_line, inv_line_vals, logs)

    def _correct_invoice_tax_amount(self, tree, invoice):
        """ The tax total may have been modified for rounding purpose, if so we should use the imported tax and not
         the computed one """
        # For each tax in our tax total, get the amount as well as the total in the xml.
        for elem in tree.findall('.//{*}TaxTotal/{*}TaxSubtotal'):
            percentage = elem.find('.//{*}TaxCategory/{*}Percent')
            if percentage is None:
                percentage = elem.find('.//{*}Percent')
            amount = elem.find('.//{*}TaxAmount')
            if (percentage is not None and percentage.text is not None) and (amount is not None and amount.text is not None):
                tax_percent = float(percentage.text)
                # Compare the result with our tax total on the invoice, and apply correction if needed.
                # First look for taxes matching the percentage in the xml.
                taxes = invoice.line_ids.tax_line_id.filtered(lambda tax: tax.amount == tax_percent)
                # If we found taxes with the correct amount, look for a tax line using it, and correct it as needed.
                if taxes:
                    tax_total = float(amount.text)
                    tax_line = invoice.line_ids.filtered(lambda line: line.tax_line_id in taxes)[:1]
                    if tax_line:
                        sign = -1 if invoice.is_inbound(include_receipts=True) else 1
                        tax_line_amount = abs(tax_line.amount_currency)
                        if abs(tax_total - tax_line_amount) <= 0.05:
                            tax_line.amount_currency = tax_total * sign

    # -------------------------------------------------------------------------
    # IMPORT : helpers
    # -------------------------------------------------------------------------

    def _get_import_document_amount_sign(self, tree):
        """
        In UBL, an invoice has tag 'Invoice' and a credit note has tag 'CreditNote'. However, a credit note can be
        expressed as an invoice with negative amounts. For this case, we need a factor to take the opposite
        of each quantity in the invoice.
        """
        if tree.tag == '{urn:oasis:names:specification:ubl:schema:xsd:Invoice-2}Invoice':
            amount_node = tree.find('.//{*}LegalMonetaryTotal/{*}TaxExclusiveAmount')
            if amount_node is not None and float(amount_node.text) < 0:
                return 'refund', -1
            return 'invoice', 1
        if tree.tag == '{urn:oasis:names:specification:ubl:schema:xsd:CreditNote-2}CreditNote':
            return 'refund', 1
        return None, None

```

## File: models\account_edi_xml_ubl_21.py

```python
# -*- coding: utf-8 -*-
from odoo import api, models


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
            'PaymentTermsType_template': 'account_edi_ubl_cii.ubl_21_PaymentTermsType',
            'CreditNoteLineType_template': 'account_edi_ubl_cii.ubl_21_CreditNoteLineType',
            'DebitNoteLineType_template': 'account_edi_ubl_cii.ubl_21_DebitNoteLineType',
            'InvoiceType_template': 'account_edi_ubl_cii.ubl_21_InvoiceType',
            'CreditNoteType_template': 'account_edi_ubl_cii.ubl_21_CreditNoteType',
            'DebitNoteType_template': 'account_edi_ubl_cii.ubl_21_DebitNoteType',
        })

        vals['vals'].update({
            'ubl_version_id': 2.1,
            'buyer_reference': invoice.commercial_partner_id.ref,
        })

        return vals

    @api.model
    def _get_customization_ids(self):
        return {
            'ubl_bis3': 'urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:billing:3.0',
            'nlcius': 'urn:cen.eu:en16931:2017#compliant#urn:fdc:nen.nl:nlcius:v1.0',
            'ubl_sg': 'urn:cen.eu:en16931:2017#conformant#urn:fdc:peppol.eu:2017:poacc:billing:international:sg:3.0',
            'xrechnung': 'urn:cen.eu:en16931:2017#compliant#urn:xeinkauf.de:kosit:xrechnung_3.0',
            'ubl_a_nz': 'urn:cen.eu:en16931:2017#conformant#urn:fdc:peppol.eu:2017:poacc:billing:international:aunz:3.0',
            'pint_jp': 'urn:peppol:pint:billing-1@jp-1',
            'pint_sg': 'urn:peppol:pint:billing-1@sg-1',
            'pint_my': 'urn:peppol:pint:billing-1@my-1',
        }

```

## File: models\account_edi_xml_ubl_a_nz.py

```python
# -*- coding: utf-8 -*-

from odoo import models


class AccountEdiXmlUBLANZ(models.AbstractModel):
    _inherit = "account.edi.xml.ubl_bis3"
    _name = 'account.edi.xml.ubl_a_nz'
    _description = "A-NZ BIS Billing 3.0"

    """
    * Documentation: https://github.com/A-NZ-PEPPOL/A-NZ-PEPPOL-BIS-3.0/tree/master/Specifications
    """

    # -------------------------------------------------------------------------
    # EXPORT
    # -------------------------------------------------------------------------

    def _export_invoice_filename(self, invoice):
        return f"{invoice.name.replace('/', '_')}_a_nz.xml"

    def _export_invoice_ecosio_schematrons(self):
        return {
            'invoice': 'eu.peppol.bis3.aunz.ubl:invoice:1.0.8',
            'credit_note': 'eu.peppol.bis3.aunz.ubl:creditnote:1.0.8',
        }

    def _get_partner_party_tax_scheme_vals_list(self, partner, role):
        # EXTENDS account.edi.xml.ubl_bis3
        vals_list = super()._get_partner_party_tax_scheme_vals_list(partner, role)

        for vals in vals_list:
            if partner.country_id.code == "AU" and partner.vat:
                vals['company_id'] = partner.vat.replace(" ", "")

        return vals_list

    def _get_partner_party_vals(self, partner, role):
        # EXTENDS account.edi.xml.ubl_bis3
        vals = super()._get_partner_party_vals(partner, role)

        if partner.country_code == 'AU' and partner.vat:
            vals['endpoint_id'] = partner.vat.replace(" ", "")
        if partner.country_code == 'NZ':
            vals['endpoint_id'] = partner.company_registry

        for party_tax_scheme in vals['party_tax_scheme_vals']:
            party_tax_scheme['tax_scheme_vals'] = {'id': 'GST'}

        return vals

    def _get_partner_party_legal_entity_vals_list(self, partner):
        # EXTENDS account.edi.xml.ubl_bis3
        vals_list = super()._get_partner_party_legal_entity_vals_list(partner)

        for vals in vals_list:
            if partner.country_code == 'AU' and partner.vat:
                vals.update({
                    'company_id': partner.vat.replace(" ", ""),
                    'company_id_attrs': {'schemeID': '0151'},
                })
            if partner.country_code == 'NZ':
                vals.update({
                    'company_id': partner.company_registry,
                    'company_id_attrs': {'schemeID': '0088'},
                })
        return vals_list

    def _get_tax_category_list(self, invoice, taxes):
        # EXTENDS account.edi.xml.ubl_bis3
        vals_list = super()._get_tax_category_list(invoice, taxes)
        for vals in vals_list:
            vals['tax_scheme_vals'] = {'id': 'GST'}
        return vals_list

    def _export_invoice_vals(self, invoice):
        # EXTENDS account.edi.xml.ubl_21
        vals = super()._export_invoice_vals(invoice)

        vals['vals'].update({
            'customization_id': self._get_customization_ids()['ubl_a_nz'],
        })

        return vals

```

## File: models\account_edi_xml_ubl_bis3.py

```python
# -*- coding: utf-8 -*-

from odoo import models, _
from odoo.addons.account_edi_ubl_cii.models.account_edi_xml_ubl_20 import UBL_NAMESPACES

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
            return [{
                'company_id': partner.peppol_endpoint,
                'tax_scheme_vals': {'id': partner.peppol_eas},
            }]

        for vals in vals_list:
            vals.pop('registration_name', None)
            vals.pop('registration_address_vals', None)

        # sources:
        #  https://anskaffelser.dev/postaward/g3/spec/current/billing-3.0/norway/#_applying_foretaksregisteret
        #  https://docs.peppol.eu/poacc/billing/3.0/bis/#national_rules (NO-R-002 (warning))
        if partner.country_id.code == "NO" and role == 'supplier':
            vals_list.append({
                'company_id': "Foretaksregisteret",
                'tax_scheme_vals': {'id': 'TAX'},
            })

        return vals_list

    def _get_partner_party_legal_entity_vals_list(self, partner):
        # EXTENDS account.edi.xml.ubl_21
        vals_list = super()._get_partner_party_legal_entity_vals_list(partner)

        for vals in vals_list:
            vals.pop('registration_address_vals', None)
            if partner.country_code == 'NL':
                vals.update({
                    'company_id': partner.peppol_endpoint,
                    'company_id_attrs': {'schemeID': partner.peppol_eas},
                })
            if partner.country_id.code == "LU":
                if 'l10n_lu_peppol_identifier' in partner._fields and partner.l10n_lu_peppol_identifier:
                    vals['company_id'] = partner.l10n_lu_peppol_identifier
                elif partner.company_registry:
                    vals['company_id'] = partner.company_registry
            if partner.country_id.code == 'DK':
                # DK-R-014: For Danish Suppliers it is mandatory to specify schemeID as "0184" (DK CVR-number) when
                # PartyLegalEntity/CompanyID is used for AccountingSupplierParty
                vals['company_id_attrs'] = {'schemeID': '0184'}
            if partner.country_code == 'SE' and partner.company_registry:
                vals['company_id'] = ''.join(char for char in partner.company_registry if char.isdigit())
            if not vals['company_id']:
                vals['company_id'] = partner.peppol_endpoint

        return vals_list

    def _get_partner_contact_vals(self, partner):
        # EXTENDS account.edi.xml.ubl_21
        vals = super()._get_partner_contact_vals(partner)

        vals.pop('id', None)

        return vals

    def _get_partner_party_vals(self, partner, role):
        # EXTENDS account.edi.xml.ubl_21
        vals = super()._get_partner_party_vals(partner, role)

        partner = partner.commercial_partner_id
        vals.update({
            'endpoint_id': partner.peppol_endpoint,
            'endpoint_id_attrs': {'schemeID': partner.peppol_eas},
        })

        return vals

    def _get_partner_party_identification_vals_list(self, partner):
        # EXTENDS account.edi.xml.ubl_21
        vals = super()._get_partner_party_identification_vals_list(partner)

        if partner.country_code == 'NL':
            vals.append({
                'id': partner.peppol_endpoint,
            })
        return vals

    def _get_delivery_vals_list(self, invoice):
        # EXTENDS account.edi.xml.ubl_21
        supplier = invoice.company_id.partner_id.commercial_partner_id
        customer = invoice.partner_id

        economic_area = self.env.ref('base.europe').country_ids.mapped('code') + ['NO']
        intracom_delivery = (customer.country_id.code in economic_area
                             and supplier.country_id.code in economic_area
                             and supplier.country_id != customer.country_id)

        # [BR-IC-12]-In an Invoice with a VAT breakdown (BG-23) where the VAT category code (BT-118) is
        # "Intra-community supply" the Deliver to country code (BT-80) shall not be blank.

        # [BR-IC-11]-In an Invoice with a VAT breakdown (BG-23) where the VAT category code (BT-118) is
        # "Intra-community supply" the Actual delivery date (BT-72) or the Invoicing period (BG-14)
        # shall not be blank.

        if intracom_delivery:
            partner_shipping = invoice.partner_shipping_id or customer

            return [{
                'actual_delivery_date': invoice.invoice_date,
                'delivery_location_vals': {
                    'delivery_address_vals': self._get_partner_address_vals(partner_shipping),
                },
            }]

        return super()._get_delivery_vals_list(invoice)

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
            vals.pop('name', None)

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
            # [UBL-CR-600] A UBL invoice should not include the InvoiceLine Item ClassifiedTaxCategory TaxExemptionReasonCode
            val.pop('tax_exemption_reason_code', None)
            # [UBL-CR-601] TaxExemptionReason must not appear in InvoiceLine Item ClassifiedTaxCategory
            # [BR-E-10] TaxExemptionReason must only appear in TaxTotal TaxSubtotal TaxCategory
            val.pop('tax_exemption_reason', None)

        return line_item_vals

    def _get_invoice_line_allowance_vals_list(self, line, tax_values_list=None):
        # EXTENDS account.edi.xml.ubl_21
        vals_list = super()._get_invoice_line_allowance_vals_list(line, tax_values_list=tax_values_list)

        for vals in vals_list:
            vals['currency_dp'] = 2

        return vals_list

    def _get_invoice_line_vals(self, line, line_id, taxes_vals):
        # EXTENDS account.edi.xml.ubl_21
        vals = super()._get_invoice_line_vals(line, line_id, taxes_vals)

        vals.pop('tax_total_vals', None)

        vals['currency_dp'] = 2
        vals['price_vals']['currency_dp'] = 2

        if line.currency_id.compare_amounts(vals['price_vals']['price_amount'], 0) == -1:
            # We can't have negative unit prices, so we invert the signs of
            # the unit price and quantity, resulting in the same amount in the end
            vals['price_vals']['price_amount'] *= -1
            vals['line_quantity'] *= -1

        return vals

    def _export_invoice_vals(self, invoice):
        # EXTENDS account.edi.xml.ubl_21
        vals = super()._export_invoice_vals(invoice)

        vals['vals'].update({
            'customization_id': self._get_customization_ids()['ubl_bis3'],
            'profile_id': 'urn:fdc:peppol.eu:2017:poacc:billing:01:1.0',
            'currency_dp': 2,
            'ubl_version_id': None,
        })
        vals['vals']['monetary_total_vals']['currency_dp'] = 2

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
            # [BR-61]-If the Payment means type code (BT-81) means SEPA credit transfer, Local credit transfer or
            # Non-SEPA international credit transfer, the Payment account identifier (BT-84) shall be present.
            # note: Payment account identifier is <cac:PayeeFinancialAccount>
            # note: no need to check account_number, because it's a required field for a partner_bank
            'cen_en16931_payment_account_identifier': self._check_required_fields(
                invoice, 'partner_bank_id'
            ) if vals['vals']['payment_means_vals_list'][0]['payment_means_code'] in (30, 58) else None,
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

        for line_vals in vals['vals']['line_vals']:
            if not line_vals['item_vals'].get('name'):
                # [BR-25]-Each Invoice line (BG-25) shall contain the Item name (BT-153).
                constraints.update({'cen_en16931_item_name': _("Each invoice line should have a product or a label.")})
                break

        for line in invoice.invoice_line_ids.filtered(lambda x: x.display_type not in ('line_note', 'line_section')):
            if len(line.tax_ids.flatten_taxes_hierarchy().filtered(lambda t: t.amount_type != 'fixed')) != 1:
                # [UBL-SR-48]-Invoice lines shall have one and only one classified tax category.
                # /!\ exception: possible to have any number of ecotaxes (fixed tax) with a regular percentage tax
                constraints.update({'cen_en16931_tax_line': _("Each invoice line shall have one and only one tax.")})

        for role in ('supplier', 'customer'):
            constraints[f'cen_en16931_{role}_country'] = self._check_required_fields(
                vals['vals'][f'accounting_{role}_party_vals']['party_vals']['postal_address_vals']['country_vals'],
                'identification_code',
                _("The country is required for the %s.", role)
            )
            scheme_vals = vals['vals'][f'accounting_{role}_party_vals']['party_vals']['party_tax_scheme_vals'][-1:]
            if (
                not (scheme_vals and scheme_vals[0]['company_id'] and scheme_vals[0]['company_id'][:2].isalpha())
                and (scheme_vals and scheme_vals[0]['tax_scheme_vals'].get('id') == 'VAT')
                and self._name in ('account.edi.xml.ubl_bis3', 'account.edi.xml.ubl_nl', 'account.edi.xml.ubl_de')
            ):
                # [BR-CO-09]-The Seller VAT identifier (BT-31), the Seller tax representative VAT identifier (BT-63)
                # and the Buyer VAT identifier (BT-48) shall have a prefix in accordance with ISO code ISO 3166-1
                # alpha-2 by which the country of issue may be identified. Nevertheless, Greece may use the prefix ‘EL’.
                constraints.update({f'cen_en16931_{role}_vat_country_code': _(
                    "The VAT of the %s should be prefixed with its country code.", role)})

        if invoice.partner_shipping_id:
            # [BR-57]-Each Deliver to address (BG-15) shall contain a Deliver to country code (BT-80).
            constraints['cen_en16931_delivery_address'] = self._check_required_fields(invoice.partner_shipping_id, 'country_id')
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
                    "%s should have a KVK or OIN number: the Peppol e-address (EAS) should be '0106' or '0190'.",
                    vals['supplier'].display_name
                ) if vals['supplier'].peppol_eas not in ('0106', '0190') else '',

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
                        "%s should have a KVK or OIN number: the Peppol e-address (EAS) should be '0106' or '0190'.",
                        vals['customer'].display_name
                    ) if vals['customer'].commercial_partner_id.peppol_eas not in ('0106', '0190') else '',
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
            })
        return constraints

    def _import_retrieve_partner_vals(self, tree, role):
        # EXTENDS account.edi.xml.ubl_20
        partner_vals = super()._import_retrieve_partner_vals(tree, role)
        endpoint_node = tree.find(f'.//cac:Accounting{role}Party/cac:Party/cbc:EndpointID', UBL_NAMESPACES)
        if endpoint_node is not None:
            peppol_eas = endpoint_node.attrib.get('schemeID')
            peppol_endpoint = endpoint_node.text
            if peppol_eas and peppol_endpoint:
                # include the EAS and endpoint in the search domain when retrieving the partner
                partner_vals.update({
                    'peppol_eas': peppol_eas,
                    'peppol_endpoint': peppol_endpoint,
                })
        return partner_vals

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

## File: models\account_edi_xml_ubl_nlcius.py

```python
# -*- coding: utf-8 -*-

from odoo import models


class AccountEdiXmlUBLNL(models.AbstractModel):
    _inherit = "account.edi.xml.ubl_bis3"
    _name = 'account.edi.xml.ubl_nl'
    _description = "SI-UBL 2.0 (NLCIUS)"

    """
    SI-UBL 2.0 (NLCIUS) and UBL Bis 3 are 2 different formats used in the Netherlands.
    (source: https://github.com/peppolautoriteit-nl/publications/tree/master/NLCIUS-PEPPOLBIS-Differences)
    NLCIUS defines a set of rules
    (source: https://www.softwarepakketten.nl/wiki_uitleg/60&bronw=7/Nadere_specificaties_EN_16931_1_norm_voor_de_Europese_kernfactuur.htm)
    Fortunately, some of these rules are already present in UBL Bis 3, but some are missing.

    For instance, Bis 3 states that the customizationID should be
    "urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:billing:3.0".
    while in NLCIUS:
    "urn:cen.eu:en16931:2017#compliant#urn:fdc:nen.nl:nlcius:v1.0".

    Bis 3 and NLCIUS are thus incompatible. Hence, the two separate formats and the additional rules in this file.

    The trick is to understand which rules are only NLCIUS specific and which are applied to the Bis 3 format.
    """

    # -------------------------------------------------------------------------
    # EXPORT
    # -------------------------------------------------------------------------

    def _export_invoice_filename(self, invoice):
        return f"{invoice.name.replace('/', '_')}_nlcius.xml"

    def _export_invoice_ecosio_schematrons(self):
        return {
            'invoice': 'org.simplerinvoicing:invoice:2.0.3.3',
            'credit_note': 'org.simplerinvoicing:creditnote:2.0.3.3',
        }

    def _get_tax_category_list(self, invoice, taxes):
        # EXTENDS account.edi.xml.ubl_bis3
        vals_list = super()._get_tax_category_list(invoice, taxes)
        for tax in vals_list:
            # [BR-NL-35] The use of a tax exemption reason code (cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory
            # /cbc:TaxExemptionReasonCode) is not recommended
            tax.pop('tax_exemption_reason_code', None)
        return vals_list

    def _get_partner_address_vals(self, partner):
        # EXTENDS account.edi.xml.ubl_bis3
        vals = super()._get_partner_address_vals(partner)
        # [BR-NL-28] The use of a country subdivision (cac:AccountingCustomerParty/cac:Party/cac:PostalAddress
        # /cbc:CountrySubentity) is not recommended
        vals.pop('country_subentity', None)
        return vals

    def _get_invoice_line_allowance_vals_list(self, line, tax_values_list=None):
        # EXTENDS account.edi.xml.ubl_bis3
        vals_list = super()._get_invoice_line_allowance_vals_list(line, tax_values_list=tax_values_list)
        # [BR-NL-32] Use of Discount reason code ( AllowanceChargeReasonCode ) is not recommended.
        # [BR-EN-34] Use of Charge reason code ( AllowanceChargeReasonCode ) is not recommended.
        # Careful! [BR-42]-Each Invoice line allowance (BG-27) shall have an Invoice line allowance reason (BT-139)
        # or an Invoice line allowance reason code (BT-140).
        for vals in vals_list:
            if vals.get('allowance_charge_reason'):
                vals.pop('allowance_charge_reason_code')
        return vals_list

    def _get_invoice_payment_means_vals_list(self, invoice):
        # EXTENDS account.edi.xml.ubl_bis3
        vals_list = super()._get_invoice_payment_means_vals_list(invoice)
        # [BR-NL-29] The use of a payment means text (cac:PaymentMeans/cbc:PaymentMeansCode/@name) is not recommended
        for vals in vals_list:
            vals.pop('payment_means_code_attrs', None)
        return vals_list

    def _export_invoice_vals(self, invoice):
        # EXTENDS account.edi.xml.ubl_bis3
        vals = super()._export_invoice_vals(invoice)

        vals['vals']['customization_id'] = self._get_customization_ids()['nlcius']

        # [BR-NL-24] Use of previous invoice date ( IssueDate ) is not recommended.
        # vals['vals'].pop('issue_date')  # careful, this causes other errors from the validator...

        return vals

```

## File: models\account_edi_xml_ubl_sg.py

```python
# -*- coding: utf-8 -*-
from odoo import models


class AccountEdiXmlUBLSG(models.AbstractModel):
    _inherit = "account.edi.xml.ubl_bis3"
    _name = "account.edi.xml.ubl_sg"
    _description = "SG BIS Billing 3.0"

    """
    Documentation: https://www.peppolguide.sg/billing/bis/
    """

    # -------------------------------------------------------------------------
    # EXPORT
    # -------------------------------------------------------------------------

    def _export_invoice_filename(self, invoice):
        return f"{invoice.name.replace('/', '_')}_sg.xml"

    def _export_invoice_ecosio_schematrons(self):
        return {
            'invoice': 'eu.peppol.bis3.sg.ubl:invoice:1.0.3',
            'credit_note': 'eu.peppol.bis3.sg.ubl:creditnote:1.0.3',
        }

    def _get_partner_party_vals(self, partner, role):
        # EXTENDS account.edi.xml.ubl_bis3
        vals = super()._get_partner_party_vals(partner, role)

        for party_tax_scheme in vals['party_tax_scheme_vals']:
            party_tax_scheme['tax_scheme_vals'] = {'id': 'GST'}

        return vals

    def _get_invoice_payment_means_vals_list(self, invoice):
        """ https://www.peppolguide.sg/billing/bis/#_payment_means_information
        """
        vals_list = super()._get_invoice_payment_means_vals_list(invoice)
        for vals in vals_list:
            vals.update({
                'payment_means_code': 54,
                'payment_means_code_attrs': {'name': 'Credit Card'},
            })

        return vals_list

    def _get_tax_sg_codes(self, invoice, tax):
        """ https://www.peppolguide.sg/billing/bis/#_gst_category_codes
        """
        tax_category_code = 'SR'
        if tax.amount == 0:
            tax_category_code = 'ZR'
        return tax_category_code

    def _get_tax_category_list(self, invoice, taxes):
        # OVERRIDE
        res = []
        for tax in taxes:
            res.append({
                'id': self._get_tax_sg_codes(invoice, tax),
                'percent': tax.amount if tax.amount_type == 'percent' else False,
                'tax_scheme_vals': {'id': 'GST'},
            })
        return res

    def _export_invoice_vals(self, invoice):
        # EXTENDS account.edi.xml.ubl_bis3
        vals = super()._export_invoice_vals(invoice)

        vals['vals'].update({
            'customization_id': self._get_customization_ids()['ubl_sg'],
        })

        return vals

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
        vals['vals']['customization_id'] = self._get_customization_ids()['xrechnung']
        return vals

    def _export_invoice_constraints(self, invoice, vals):
        # EXTENDS account.edi.xml.ubl_bis3
        constraints = super()._export_invoice_constraints(invoice, vals)

        constraints.update({
            'bis3_de_supplier_telephone_required': self._check_required_fields(vals['supplier'], ['phone', 'mobile']),
            'bis3_de_supplier_electronic_mail_required': self._check_required_fields(vals['supplier'], 'email'),
        })

        return constraints

    def _get_partner_party_vals(self, partner, role):
        # EXTENDS account.edi.xml.ubl_bis3
        vals = super()._get_partner_party_vals(partner, role)

        if not vals.get('endpoint_id') and partner.email:
            vals.update({
                'endpoint_id': partner.email,
                'endpoint_id_attrs': {'schemeID': 'EM'},
            })

        return vals

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
from odoo import api, fields, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    ubl_cii_xml_id = fields.Many2one(
        comodel_name='ir.attachment',
        string="Attachment",
        compute=lambda self: self._compute_linked_attachment_id('ubl_cii_xml_id', 'ubl_cii_xml_file'),
        depends=['ubl_cii_xml_id']
    )
    ubl_cii_xml_file = fields.Binary(
        attachment=True,
        string="UBL/CII File",
        copy=False,
    )

    # -------------------------------------------------------------------------
    # EDI
    # -------------------------------------------------------------------------

    @api.model
    def _get_ubl_cii_builder_from_xml_tree(self, tree):
        customization_id = tree.find('{*}CustomizationID')
        if tree.tag == '{urn:un:unece:uncefact:data:standard:CrossIndustryInvoice:100}CrossIndustryInvoice':
            return self.env['account.edi.xml.cii']
        ubl_version = tree.find('{*}UBLVersionID')
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
            if customization_id.text == 'urn:cen.eu:en16931:2017#conformant#urn:fdc:peppol.eu:2017:poacc:billing:international:aunz:3.0':
                return self.env['account.edi.xml.ubl_a_nz']
            if customization_id.text == 'urn:cen.eu:en16931:2017#conformant#urn:fdc:peppol.eu:2017:poacc:billing:international:sg:3.0':
                return self.env['account.edi.xml.ubl_sg']
            if 'urn:cen.eu:en16931:2017' in customization_id.text:
                return self.env['account.edi.xml.ubl_bis3']

    def _get_edi_decoder(self, file_data, new=False):
        # EXTENDS 'account'
        if file_data['type'] == 'xml':
            ubl_cii_xml_builder = self._get_ubl_cii_builder_from_xml_tree(file_data['xml_tree'])
            if ubl_cii_xml_builder is not None:
                return ubl_cii_xml_builder._import_invoice_ubl_cii

        return super()._get_edi_decoder(file_data, new=new)

    def _need_ubl_cii_xml(self):
        self.ensure_one()
        return not self.invoice_pdf_report_id \
            and not self.ubl_cii_xml_id \
            and self.is_sale_document() \
            and bool(self.partner_id.commercial_partner_id.ubl_cii_format)

```

## File: models\ir_actions_report.py

```python
import io

from odoo import models


class IrActionsReport(models.Model):
    _inherit = 'ir.actions.report'

    def _render_qweb_pdf_prepare_streams(self, report_ref, data, res_ids=None):
        # EXTENDS base
        collected_streams = super()._render_qweb_pdf_prepare_streams(report_ref, data, res_ids)

        # allows to add factur-x.xml to custom PDF templates (comma separated list of template names)
        custom_templates = self.env['ir.config_parameter'].sudo().get_param('account.custom_templates_facturx_list', '')
        custom_templates = [report.strip() for report in custom_templates.split(',')]

        if (
            collected_streams
            and res_ids
            and len(res_ids) == 1
            and (
                # only triggered from the "Print" action for default template
                self._is_invoice_report(report_ref) and not self.env.context.get('from_account_move_send')
                # triggered from "Print" and "Send & Print" actions for custom templates
                or self._get_report(report_ref).report_name in custom_templates
            )
        ):
            # Generate and embed Factur-X
            invoice = self.env['account.move'].browse(res_ids)
            if invoice.is_sale_document() and invoice.state == 'posted':
                pdf_stream = collected_streams[invoice.id]['stream']
                invoice_data = {'pdf_attachment_values': {'raw': pdf_stream.getvalue()}}
                self.env['account.move.send']._hook_invoice_document_after_pdf_report_render(invoice, invoice_data)
                collected_streams[invoice.id]['stream'] = io.BytesIO(invoice_data['pdf_attachment_values']['raw'])
        return collected_streams

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class ResCompany(models.Model):
    _inherit = 'res.company'

    invoice_is_ubl_cii = fields.Boolean('Generate Peppol format by default', default=False)

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    invoice_is_ubl_cii = fields.Boolean(string='Peppol format', related='company_id.invoice_is_ubl_cii', readonly=False)

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from stdnum.fr import siret

from odoo import models, fields, api, _
from odoo.exceptions import ValidationError
from odoo.addons.account_edi_ubl_cii.models.account_edi_common import EAS_MAPPING
from odoo.addons.account.models.company import PEPPOL_DEFAULT_COUNTRIES


class ResPartner(models.Model):
    _inherit = 'res.partner'

    ubl_cii_format = fields.Selection(
        string="Format",
        selection=[
            ('facturx', "Factur-X (CII)"),
            ('ubl_bis3', "BIS Billing 3.0"),
            ('xrechnung', "XRechnung CIUS"),
            ('nlcius', "NLCIUS"),
            ('ubl_a_nz', "BIS Billing 3.0 A-NZ"),
            ('ubl_sg', "BIS Billing 3.0 SG"),
        ],
        compute='_compute_ubl_cii_format',
        store=True,
        readonly=False,
    )
    peppol_endpoint = fields.Char(
        string="Peppol Endpoint",
        help="Unique identifier used by the BIS Billing 3.0 and its derivatives, also known as 'Endpoint ID'.",
        compute="_compute_peppol_endpoint",
        store=True,
        readonly=False,
        tracking=True,
    )
    peppol_eas = fields.Selection(
        string="Peppol e-address (EAS)",
        help="""Code used to identify the Endpoint for BIS Billing 3.0 and its derivatives.
             List available at https://docs.peppol.eu/poacc/billing/3.0/codelist/eas/""",
        compute="_compute_peppol_eas",
        store=True,
        readonly=False,
        tracking=True,
        selection=[
            ('0002', "0002 - System Information et Repertoire des Entreprise et des Etablissements: SIRENE"),
            ('0007', "0007 - Organisationsnummer (Swedish legal entities)"),
            ('0009', "0009 - SIRET-CODE"),
            ('0037', "0037 - LY-tunnus"),
            ('0060', "0060 - Data Universal Numbering System (D-U-N-S Number)"),
            ('0088', "0088 - EAN Location Code"),
            ('0096', "0096 - DANISH CHAMBER OF COMMERCE Scheme (EDIRA compliant)"),
            ('0097', "0097 - FTI - Ediforum Italia, (EDIRA compliant)"),
            ('0106', "0106 - Association of Chambers of Commerce and Industry in the Netherlands, (EDIRA compliant)"),
            ('0130', "0130 - Directorates of the European Commission"),
            ('0135', "0135 - SIA Object Identifiers"),
            ('0142', "0142 - SECETI Object Identifiers"),
            ('0151', "0151 - Australian Business Number (ABN) Scheme"),
            ('0183', "0183 - Swiss Unique Business Identification Number (UIDB)"),
            ('0184', "0184 - DIGSTORG"),
            ('0188', "0188 - Corporate Number of The Social Security and Tax Number System"),
            ('0190', "0190 - Dutch Originator's Identification Number"),
            ('0191', "0191 - Centre of Registers and Information Systems of the Ministry of Justice"),
            ('0192', "0192 - Enhetsregisteret ved Bronnoysundregisterne"),
            ('0193', "0193 - UBL.BE party identifier"),
            ('0195', "0195 - Singapore UEN identifier"),
            ('0196', "0196 - Kennitala - Iceland legal id for individuals and legal entities"),
            ('0198', "0198 - ERSTORG"),
            ('0199', "0199 - Legal Entity Identifier (LEI)"),
            ('0200', "0200 - Legal entity code (Lithuania)"),
            ('0201', "0201 - Codice Univoco Unità Organizzativa iPA"),
            ('0202', "0202 - Indirizzo di Posta Elettronica Certificata"),
            ('0204', "0204 - Leitweg-ID"),
            ('0208', "0208 - Numero d'entreprise / ondernemingsnummer / Unternehmensnummer"),
            ('0209', "0209 - GS1 identification keys"),
            ('0210', "0210 - CODICE FISCALE"),
            ('0211', "0211 - PARTITA IVA"),
            ('0212', "0212 - Finnish Organization Identifier"),
            ('0213', "0213 - Finnish Organization Value Add Tax Identifier"),
            ('0215', "0215 - Net service ID"),
            ('0216', "0216 - OVTcode"),
            ('0221', "0221 - The registered number of the qualified invoice issuer (Japan)"),
            ('0230', "0230 - National e-Invoicing Framework (Malaysia)"),
            ('9901', "9901 - Danish Ministry of the Interior and Health"),
            ('9910', "9910 - Hungary VAT number"),
            ('9913', "9913 - Business Registers Network"),
            ('9914', "9914 - Österreichische Umsatzsteuer-Identifikationsnummer"),
            ('9915', "9915 - Österreichisches Verwaltungs bzw. Organisationskennzeichen"),
            ('9918', "9918 - SOCIETY FOR WORLDWIDE INTERBANK FINANCIAL, TELECOMMUNICATION S.W.I.F.T"),
            ('9919', "9919 - Kennziffer des Unternehmensregisters"),
            ('9920', "9920 - Agencia Española de Administración Tributaria"),
            ('9922', "9922 - Andorra VAT number"),
            ('9923', "9923 - Albania VAT number"),
            ('9924', "9924 - Bosnia and Herzegovina VAT number"),
            ('9925', "9925 - Belgium VAT number"),
            ('9926', "9926 - Bulgaria VAT number"),
            ('9927', "9927 - Switzerland VAT number"),
            ('9928', "9928 - Cyprus VAT number"),
            ('9929', "9929 - Czech Republic VAT number"),
            ('9930', "9930 - Germany VAT number"),
            ('9931', "9931 - Estonia VAT number"),
            ('9932', "9932 - United Kingdom VAT number"),
            ('9933', "9933 - Greece VAT number"),
            ('9934', "9934 - Croatia VAT number"),
            ('9935', "9935 - Ireland VAT number"),
            ('9936', "9936 - Liechtenstein VAT number"),
            ('9937', "9937 - Lithuania VAT number"),
            ('9938', "9938 - Luxemburg VAT number"),
            ('9939', "9939 - Latvia VAT number"),
            ('9940', "9940 - Monaco VAT number"),
            ('9941', "9941 - Montenegro VAT number"),
            ('9942', "9942 - Macedonia, the former Yugoslav Republic of VAT number"),
            ('9943', "9943 - Malta VAT number"),
            ('9944', "9944 - Netherlands VAT number"),
            ('9945', "9945 - Poland VAT number"),
            ('9946', "9946 - Portugal VAT number"),
            ('9947', "9947 - Romania VAT number"),
            ('9948', "9948 - Serbia VAT number"),
            ('9949', "9949 - Slovenia VAT number"),
            ('9950', "9950 - Slovakia VAT number"),
            ('9951', "9951 - San Marino VAT number"),
            ('9952', "9952 - Türkiye VAT number"),
            ('9953', "9953 - Holy See (Vatican City State) VAT number"),
            ('9955', "9955 - Swedish VAT number"),
            ('9957', "9957 - French VAT number"),
            ('9959', "9959 - Employer Identification Number (EIN, USA)"),
        ]
    )
    hide_peppol_fields = fields.Boolean(compute='_compute_hide_peppol_fields')

    @api.constrains('peppol_eas')
    def _check_peppol_eas(self):
        for partner in self:
            if partner.peppol_eas in ('0037', '0212', '0213', '0215'):
                raise ValidationError(_("Peppol EAS codes 0037, 0212, 0213, 0215 are deprecated. Please use 0216 instead."))
            elif partner.peppol_eas == '9955':
                raise ValidationError(_("Peppol EAS code 9955 is deprecated. Please use 0007 instead."))
            elif partner.peppol_eas == '9901':
                raise ValidationError(_("Peppol EAS code 9901 is deprecated. Please use a different Danish EAS code instead."))

    @api.constrains('peppol_endpoint')
    def _check_peppol_fields(self):
        for partner in self:
            if partner.peppol_endpoint and partner.peppol_eas:
                error = self._build_error_peppol_endpoint(partner.peppol_eas, partner.peppol_endpoint)
                if error:
                    raise ValidationError(error)

    @api.model
    def _get_ubl_cii_formats(self):
        return {
            'DE': 'xrechnung',
            'AU': 'ubl_a_nz',
            'NZ': 'ubl_a_nz',
            'NL': 'nlcius',
            'FR': 'facturx',
            'SG': 'ubl_sg',
        }

    def _peppol_eas_endpoint_depends(self):
        # field dependencies of methods _compute_peppol_endpoint() and _compute_peppol_eas()
        # because we need to extend depends in l10n modules
        return ['country_code', 'vat', 'company_registry']

    @api.depends(lambda self: self._peppol_eas_endpoint_depends())
    def _compute_ubl_cii_format(self):
        format_mapping = self._get_ubl_cii_formats()
        for partner in self:
            country_code = partner._deduce_country_code()
            if country_code in format_mapping:
                partner.ubl_cii_format = format_mapping[country_code]
            elif country_code in PEPPOL_DEFAULT_COUNTRIES and country_code in EAS_MAPPING:
                partner.ubl_cii_format = 'ubl_bis3'
            else:
                partner.ubl_cii_format = partner.ubl_cii_format

    @api.depends(lambda self: self._peppol_eas_endpoint_depends() + ['peppol_eas'])
    def _compute_peppol_endpoint(self):
        """ If the EAS changes and a valid endpoint is available, set it. Otherwise, keep the existing value."""
        for partner in self:
            partner.peppol_endpoint = partner.peppol_endpoint
            country_code = partner._deduce_country_code()
            if country_code in EAS_MAPPING:
                field = EAS_MAPPING[country_code].get(partner.peppol_eas)
                if field \
                        and field in partner._fields \
                        and partner[field] \
                        and not partner._build_error_peppol_endpoint(partner.peppol_eas, partner[field]):
                    partner.peppol_endpoint = partner[field]

    @api.depends(lambda self: self._peppol_eas_endpoint_depends())
    def _compute_peppol_eas(self):
        """
        If the country_code changes, recompute the EAS only if there is a country_code, it exists in the
        EAS_MAPPING, and the current EAS is not consistent with the new country_code.
        """
        for partner in self:
            partner.peppol_eas = partner.peppol_eas
            country_code = partner._deduce_country_code()
            if country_code in EAS_MAPPING:
                eas_to_field = EAS_MAPPING[country_code]
                if partner.peppol_eas not in eas_to_field.keys():
                    new_eas = next(iter(EAS_MAPPING[country_code].keys()))
                    # Iterate on the possible EAS until a valid one is found
                    for eas, field in eas_to_field.items():
                        if field and field in partner._fields and partner[field]:
                            if not partner._build_error_peppol_endpoint(eas, partner[field]):
                                new_eas = eas
                                break
                    partner.peppol_eas = new_eas

    @api.depends('ubl_cii_format')
    def _compute_hide_peppol_fields(self):
        """ Hides the people fields depending on the UBL format. Can be extended to add different hiding conditions. """
        for partner in self:
            partner.hide_peppol_fields = not partner.ubl_cii_format or partner.ubl_cii_format == 'facturx'

    def _build_error_peppol_endpoint(self, eas, endpoint):
        """ This function contains all the rules regarding the peppol_endpoint."""
        if eas == '0208' and not re.match(r"^\d{10}$", endpoint):
            return _("The Peppol endpoint is not valid. The expected format is: 0239843188")
        if eas == '0009' and not siret.is_valid(endpoint):
            return _("The Peppol endpoint is not valid. The expected format is: 73282932000074")
        if eas == '0007' and not re.match(r"^\d{10}$", endpoint):
            return _("The Peppol endpoint is not valid. "
                     "It should contain exactly 10 digits (Company Registry number)."
                     "The expected format is: 1234567890")

    def _get_edi_builder(self):
        self.ensure_one()
        if self.ubl_cii_format == 'xrechnung':
            return self.env['account.edi.xml.ubl_de']
        if self.ubl_cii_format == 'facturx':
            return self.env['account.edi.xml.cii']
        if self.ubl_cii_format == 'ubl_a_nz':
            return self.env['account.edi.xml.ubl_a_nz']
        if self.ubl_cii_format == 'nlcius':
            return self.env['account.edi.xml.ubl_nl']
        if self.ubl_cii_format == 'ubl_bis3':
            return self.env['account.edi.xml.ubl_bis3']
        if self.ubl_cii_format == 'ubl_sg':
            return self.env['account.edi.xml.ubl_sg']

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import account_edi_common
from . import account_edi_xml_cii_facturx
from . import account_edi_xml_ubl_20
from . import account_edi_xml_ubl_21
from . import account_edi_xml_ubl_bis3
from . import account_edi_xml_ubl_xrechnung
from . import account_edi_xml_ubl_nlcius
from . import account_edi_xml_ubl_efff
from . import account_edi_xml_ubl_a_nz
from . import account_edi_xml_ubl_sg
from . import account_move
from . import res_partner
from . import res_company
from . import res_config_settings
from . import ir_actions_report

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.account_edi_ubl_cii</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//setting[@id='default_setting_options']/div/div" position="inside">
                <div>
                    <field name="invoice_is_ubl_cii"/>
                    <label for="invoice_is_ubl_cii"/>
                    <span class="fa fa-lg fa-building-o" title="Values set here are company-specific."/>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<odoo>
    <record id="view_partner_property_form" model="ir.ui.view">
        <field name="name">res.partner.property.form.inherit</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='banks']" position="after">
                <group string="Electronic Invoicing" name="electronic_invoicing">
                    <field name="hide_peppol_fields" invisible="True"/>
                    <field name="ubl_cii_format" widget="selection"/>
                    <field name="peppol_eas" widget="selection"
                           invisible="hide_peppol_fields"/>
                    <field name="peppol_endpoint"
                           invisible="hide_peppol_fields"/>
                </group>
            </xpath>
        </field>
    </record>

    <record id="res_partner_view_search" model="ir.ui.view">
        <field name="name">res.partner.search.inherit</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_res_partner_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='group_by']" position="after">
                <filter name="ubl_cii_format" string="EDI Format" context="{'group_by':'ubl_cii_format'}"/>
            </xpath>
        </field>
    </record>

    <record id="res_partner_view_tree" model="ir.ui.view">
        <field name="name">res.partner.tree.inherit</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="after">
                <field name="ubl_cii_format" string="EDI Format" optional="hide"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\account_move_send.py

```python
# -*- coding: utf-8 -*-
import base64
import logging
import io

from lxml import etree
from xml.sax.saxutils import escape, quoteattr

from odoo import _, api, fields, models, tools, SUPERUSER_ID
from odoo.tools import cleanup_xml_node
from odoo.tools.pdf import OdooPdfFileReader, OdooPdfFileWriter

_logger = logging.getLogger(__name__)


class AccountMoveSend(models.TransientModel):
    _inherit = 'account.move.send'

    enable_ubl_cii_xml = fields.Boolean(compute='_compute_enable_ubl_cii_xml')
    checkbox_ubl_cii_label = fields.Char(compute='_compute_checkbox_ubl_cii_label')
    checkbox_ubl_cii_xml = fields.Boolean(compute='_compute_checkbox_ubl_cii_xml', store=True, readonly=False)
    ubl_partner_warning = fields.Char(
        string="Partner warning",
        compute="_compute_ubl_warnings",
    )
    show_ubl_company_warning = fields.Boolean(
        string="Company warning",
        compute="_compute_ubl_warnings",
    )

    def _get_wizard_values(self):
        # EXTENDS 'account'
        values = super()._get_wizard_values()
        values['ubl_cii_xml'] = self.checkbox_ubl_cii_xml
        return values

    @api.model
    def _get_wizard_vals_restrict_to(self, only_options):
        # EXTENDS 'account'
        values = super()._get_wizard_vals_restrict_to(only_options)
        return {
            'checkbox_ubl_cii_xml': False,
            **values,
        }

    # -------------------------------------------------------------------------
    # COMPUTE METHODS
    # -------------------------------------------------------------------------

    @api.depends('move_ids')
    def _compute_checkbox_ubl_cii_label(self):
        for wizard in self:
            wizard.checkbox_ubl_cii_label = False
            if wizard.mode in ('invoice_single', 'invoice_multi'):
                code_to_label = dict(wizard.move_ids.partner_id._fields['ubl_cii_format'].selection)
                codes = wizard.move_ids.partner_id.commercial_partner_id.mapped('ubl_cii_format')
                if any(codes):
                    wizard.checkbox_ubl_cii_label = ", ".join(code_to_label[c] for c in set(codes) if c)

    @api.depends('move_ids')
    def _compute_enable_ubl_cii_xml(self):
        for wizard in self:
            wizard.enable_ubl_cii_xml = any(m._need_ubl_cii_xml() for m in wizard.move_ids)

    @api.depends('checkbox_ubl_cii_xml')
    def _compute_mail_attachments_widget(self):
        # EXTENDS 'account' - add depends
        super()._compute_mail_attachments_widget()

    @api.depends('enable_ubl_cii_xml')
    def _compute_checkbox_ubl_cii_xml(self):
        for wizard in self:
            wizard.checkbox_ubl_cii_xml = wizard.enable_ubl_cii_xml and (wizard.checkbox_ubl_cii_xml or wizard.company_id.invoice_is_ubl_cii)

    @api.depends('move_ids')
    def _compute_ubl_warnings(self):
        for wizard in self:
            wizard.show_ubl_company_warning = False
            wizard.ubl_partner_warning = False
            if not set(wizard.move_ids.partner_id.commercial_partner_id.mapped('ubl_cii_format')) - {False, 'facturx', 'oioubl_201'}:
                return

            wizard.show_ubl_company_warning = not (wizard.company_id.partner_id.peppol_eas and wizard.company_id.partner_id.peppol_endpoint)
            not_configured_partners = wizard.move_ids.partner_id.commercial_partner_id.filtered(
                lambda partner: not (partner.peppol_eas and partner.peppol_endpoint)
            )
            if len(not_configured_partners) == 1:
                wizard.ubl_partner_warning = _("This partner is missing Peppol EAS or Peppol Endpoint field. "
                                        "Please check those in its Accounting tab or the generated file will be incomplete.")
            if len(not_configured_partners) > 1:
                names = ', '.join(not_configured_partners[:5].mapped('display_name'))
                wizard.ubl_partner_warning = _("The following partners are missing Peppol EAS or Peppol Endpoint field: %s. "
                                        "Please check those in their Accounting tab. "
                                        "Otherwise, the generated files will be incomplete.", names)

    # -------------------------------------------------------------------------
    # ATTACHMENTS
    # -------------------------------------------------------------------------

    @api.model
    def _get_invoice_extra_attachments(self, move):
        # EXTENDS 'account'
        return super()._get_invoice_extra_attachments(move) + move.ubl_cii_xml_id

    def _needs_ubl_cii_placeholder(self):
        return self.enable_ubl_cii_xml and self.checkbox_ubl_cii_xml

    def _get_placeholder_mail_attachments_data(self, move):
        # EXTENDS 'account'
        results = super()._get_placeholder_mail_attachments_data(move)

        if self.mode == 'invoice_single' and self._needs_ubl_cii_placeholder():
            builder = move.partner_id.commercial_partner_id._get_edi_builder()
            filename = builder._export_invoice_filename(move)
            results.append({
                'id': f'placeholder_{filename}',
                'name': filename,
                'mimetype': 'application/xml',
                'placeholder': True,
            })

        return results

    # -------------------------------------------------------------------------
    # BUSINESS ACTIONS
    # -------------------------------------------------------------------------

    @api.model
    def _hook_invoice_document_before_pdf_report_render(self, invoice, invoice_data):
        # EXTENDS 'account'
        super()._hook_invoice_document_before_pdf_report_render(invoice, invoice_data)

        if invoice_data.get('ubl_cii_xml') and invoice._need_ubl_cii_xml():
            builder = invoice.partner_id.commercial_partner_id._get_edi_builder()
            xml_content, errors = builder._export_invoice(invoice)
            filename = builder._export_invoice_filename(invoice)

            # Failed.
            if errors:
                invoice_data['error'] = {
                    'error_title': _("Errors occurred while creating the EDI document (format: %s):", builder._description),
                    'errors': errors,
                }
                invoice_data['error_but_continue'] = True
            else:
                invoice_data['ubl_cii_xml_attachment_values'] = {
                    'name': filename,
                    'raw': xml_content,
                    'mimetype': 'application/xml',
                    'res_model': invoice._name,
                    'res_id': invoice.id,
                    'res_field': 'ubl_cii_xml_file',  # Binary field
                }
                invoice_data['ubl_cii_xml_options'] = {
                    'ubl_cii_format': invoice.partner_id.commercial_partner_id.ubl_cii_format,
                    'builder': builder,
                }

    @api.model
    def _hook_invoice_document_after_pdf_report_render(self, invoice, invoice_data):
        # EXTENDS 'account'
        super()._hook_invoice_document_after_pdf_report_render(invoice, invoice_data)

        # Add PDF to XML
        if 'ubl_cii_xml_options' in invoice_data and invoice_data['ubl_cii_xml_options']['ubl_cii_format'] != 'facturx':
            self._postprocess_invoice_ubl_xml(invoice, invoice_data)

        # Always silently generate a Factur-X and embed it inside the PDF for inter-portability
        if invoice_data.get('ubl_cii_xml_options', {}).get('ubl_cii_format') == 'facturx':
            xml_facturx = invoice_data['ubl_cii_xml_attachment_values']['raw']
        else:
            xml_facturx = self.env['account.edi.xml.cii']._export_invoice(invoice)[0]

        # during tests, no wkhtmltopdf, create the attachment for test purposes
        if tools.config['test_enable']:
            self.env['ir.attachment'].sudo().create({
                'name': 'factur-x.xml',
                'raw': xml_facturx,
                'res_id': invoice.id,
                'res_model': 'account.move',
            })
            return

        # Read pdf content.
        pdf_values = invoice_data.get('pdf_attachment_values') or invoice_data['proforma_pdf_attachment_values']
        reader_buffer = io.BytesIO(pdf_values['raw'])
        reader = OdooPdfFileReader(reader_buffer, strict=False)

        # Post-process.
        writer = OdooPdfFileWriter()
        writer.cloneReaderDocumentRoot(reader)

        writer.addAttachment('factur-x.xml', xml_facturx, subtype='text/xml')

        # PDF-A.
        if invoice_data.get('ubl_cii_xml_options', {}).get('ubl_cii_format') == 'facturx' \
                and not writer.is_pdfa:
            try:
                writer.convert_to_pdfa()
            except Exception as e:
                _logger.exception("Error while converting to PDF/A: %s", e)

            # Extra metadata to be Factur-x PDF-A compliant.
            content = self.env['ir.qweb']._render(
                'account_edi_ubl_cii.account_invoice_pdfa_3_facturx_metadata',
                {
                    'title': invoice.name,
                    'date': fields.Date.context_today(self),
                },
            )
            writer.add_file_metadata(content.encode())

        # Replace the current content.
        writer_buffer = io.BytesIO()
        writer.write(writer_buffer)
        pdf_values['raw'] = writer_buffer.getvalue()
        reader_buffer.close()
        writer_buffer.close()

    @api.model
    def _postprocess_invoice_ubl_xml(self, invoice, invoice_data):
        # Adding the PDF to the XML
        tree = etree.fromstring(invoice_data['ubl_cii_xml_attachment_values']['raw'])
        anchor_elements = tree.xpath("//*[local-name()='AccountingSupplierParty']")
        if not anchor_elements:
            return

        xmlns_move_type = 'Invoice' if invoice.move_type == 'out_invoice' else 'CreditNote'
        pdf_values = invoice_data.get('pdf_attachment_values') or invoice_data['proforma_pdf_attachment_values']
        filename = pdf_values['name']
        content = pdf_values['raw']

        doc_type_node = ""
        edi_model = invoice_data["ubl_cii_xml_options"]["builder"]
        doc_type_code_vals = edi_model._get_document_type_code_vals(invoice, invoice_data)
        if doc_type_code_vals['value']:
            doc_type_code_attrs = " ".join(f'{name}="{value}"' for name, value in doc_type_code_vals['attrs'].items())
            doc_type_node = f"<cbc:DocumentTypeCode {doc_type_code_attrs}>{doc_type_code_vals['value']}</cbc:DocumentTypeCode>"
        to_inject = f'''
            <cac:AdditionalDocumentReference
                xmlns="urn:oasis:names:specification:ubl:schema:xsd:{xmlns_move_type}-2"
                xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                <cbc:ID>{escape(filename)}</cbc:ID>
                {doc_type_node}
                <cac:Attachment>
                    <cbc:EmbeddedDocumentBinaryObject
                        mimeCode="application/pdf"
                        filename={quoteattr(filename)}>
                        {base64.b64encode(content).decode()}
                    </cbc:EmbeddedDocumentBinaryObject>
                </cac:Attachment>
            </cac:AdditionalDocumentReference>
        '''

        anchor_index = tree.index(anchor_elements[0])
        tree.insert(anchor_index, etree.fromstring(to_inject))
        invoice_data['ubl_cii_xml_attachment_values']['raw'] = etree.tostring(
            cleanup_xml_node(tree), xml_declaration=True, encoding='UTF-8'
        )

    @api.model
    def _link_invoice_documents(self, invoice, invoice_data):
        # EXTENDS 'account'
        super()._link_invoice_documents(invoice, invoice_data)

        attachment_vals = invoice_data.get('ubl_cii_xml_attachment_values')
        if attachment_vals:
            self.env['ir.attachment'].with_user(SUPERUSER_ID).create(attachment_vals)
            invoice.invalidate_recordset(fnames=['ubl_cii_xml_id', 'ubl_cii_xml_file'])

```

## File: wizard\account_move_send_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="account_move_send_form" model="ir.ui.view">
        <field name="name">account.move.send.form</field>
        <field name="model">account.move.send</field>
        <field name="inherit_id" ref="account.account_move_send_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='warnings']" position="inside">
                <field name="show_ubl_company_warning" invisible="1"/>
                <div class="alert alert-info"
                     role="alert"
                     invisible="not checkbox_ubl_cii_xml or not ubl_partner_warning">
                    <field name="ubl_partner_warning"/>
                </div>
                <div class="alert alert-info"
                     role="alert"
                     invisible="not checkbox_ubl_cii_xml or not show_ubl_company_warning">
                    Please fill in Peppol EAS and Peppol Endpoint in your company form to generate a complete file.
                </div>
            </xpath>
            <xpath expr="//div[@name='advanced_options']" position="inside">
                <field name="enable_ubl_cii_xml" invisible="1"/>
                <div name="option_xml"
                     invisible="not enable_ubl_cii_xml">
                    <!-- Use one field as the label for another -->
                    <field name="checkbox_ubl_cii_xml" nolabel="1" style="display: inline-block"/>
                    <field name="checkbox_ubl_cii_label" style="position: absolute"/>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: wizard\__init__.py

```python
from . import account_move_send

```

