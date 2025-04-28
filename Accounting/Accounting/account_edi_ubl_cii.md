# Odoo Module: account_edi_ubl_cii

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
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
        'views/res_partner_views.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'account_edi_ubl_cii/static/src/scss/**/*',
        ],
    },
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
                        <ram:Name t-out="line.product_id.name"/>
                        <ram:Description t-out="line.name" t-if="line.name != line.product_id.name"/>
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
        <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
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
                <cbc:Name t-att="country_vals.get('name_attrs')" t-out="country_vals.get('name')"/>
            </cac:Country>
        </t>
    </template>

    <template id="ubl_20_PartyType">
        <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
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
            <cac:PhysicalLocation>
                <t t-set="physical_location_vals" t-value="vals.get('physical_location_vals', {})"/>
                <cac:Address>
                    <t t-call="{{AddressType_template}}">
                        <t t-set="vals" t-value="physical_location_vals.get('address_vals', {})"/>
                    </t>
                </cac:Address>
            </cac:PhysicalLocation>
            <cac:PartyTaxScheme t-foreach="vals.get('party_tax_scheme_vals', [])" t-as="foreach_vals">
                <cbc:RegistrationName t-out="foreach_vals.get('registration_name')"/>
                <cbc:CompanyID
                    t-att="foreach_vals.get('company_id_attrs', {})"
                    t-out="foreach_vals.get('company_id')"/>
                <cbc:TaxLevelCode
                    t-att="foreach_vals.get('tax_level_code_attrs', {})"
                    t-out="foreach_vals.get('tax_level_code')"/>
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
        <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
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
        <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
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
        <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
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
                <cbc:PerUnitAmount
                    t-att-currencyID="foreach_vals['currency'].name"
                    t-out="foreach_vals.get('per_unit_amount')"/>
                <cac:TaxCategory>
                    <t t-call="{{TaxCategoryType_template}}">
                        <t t-set="vals" t-value="foreach_vals.get('tax_category_vals', {})"/>
                    </t>
                </cac:TaxCategory>
            </cac:TaxSubtotal>
        </t>
    </template>

    <template id="ubl_20_AllowanceChargeType">
        <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
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
            <cbc:ReferenceID t-out="foreach_vals.get('reference_id')"/>
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

    <template id="ubl_20_InvoicePeriodType">
        <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
            <cbc:StartDate t-out="vals.get('start_date')"/>
            <cbc:StartTime t-out="vals.get('start_time')"/>
            <cbc:EndDate t-out="vals.get('end_date')"/>
            <cbc:EndTime t-out="vals.get('end_time')"/>
            <cbc:DurationMeasure t-out="vals.get('duration_measure')"/>
            <cbc:DescriptionCode t-out="vals.get('description_code')"/>
            <cbc:Description t-out="vals.get('description')"/>
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
        <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
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
                <cbc:BrandName t-out="item_vals.get('brand_name')"/>
                <cbc:ModelName t-out="item_vals.get('model_name')"/>
                <cac:SellersItemIdentification>
                    <t t-set="sellers_item_identification_vals" t-value="item_vals.get('sellers_item_identification_vals', {})"/>
                    <cbc:ID t-out="sellers_item_identification_vals.get('id')"/>
                    <cbc:ExtendedID t-out="sellers_item_identification_vals.get('extended_id')"/>
                </cac:SellersItemIdentification>
                <cac:StandardItemIdentification>
                    <t t-set="standard_item_identification_vals" t-value="item_vals.get('standard_item_identification_vals', {})"/>
                    <cbc:ID t-att="standard_item_identification_vals.get('id_attrs')"
                            t-out="standard_item_identification_vals.get('id')"/>
                </cac:StandardItemIdentification>
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

    <template id="ubl_20_InvoiceLineType" inherit_id="account_edi_ubl_cii.ubl_20_CommonLineType" primary="True">
        <xpath expr="//*[local-name()='Note']" position="after">
            <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:InvoicedQuantity
                    t-att="vals.get('line_quantity_attrs', {})"
                    t-out="vals.get('line_quantity')"/>
            </t>
        </xpath>
        <xpath expr="//*[local-name()='LineExtensionAmount']" position="after">
            <cbc:FreeOfChargeIndicator xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                t-out="vals.get('free_of_charge_indicator')"/>
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
            <cbc:UUID
                t-att="vals.get('uuid_attrs', {})"
                t-out="vals.get('uuid')"/>
            <cbc:IssueDate t-out="vals.get('issue_date')"/>
            <cbc:IssueTime t-out="vals.get('issue_time')"/>
            <cbc:Note t-foreach="vals.get('note_vals', [])" t-as="note_vals"
                t-att="note_vals.get('note_attrs', {})"
                t-out="note_vals.get('note')"/>
            <cbc:DocumentCurrencyCode
                t-att="vals.get('document_currency_code_attrs', {})"
                t-out="vals.get('document_currency_code')"/>
            <cbc:TaxCurrencyCode
                t-att="vals.get('tax_currency_code_attrs', {})"
                t-out="vals.get('tax_currency_code')"/>
            <cbc:LineCountNumeric t-out="vals.get('line_count_numeric')"/>
            <cac:InvoicePeriod t-foreach="vals.get('invoice_period_vals_list', [])" t-as="foreach_vals">
                <t t-call="{{InvoicePeriodType_template}}">
                    <t t-set="vals" t-value="foreach_vals"/>
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
                    <cbc:UUID
                        t-att="billing_reference_vals.get('uuid_attrs')"
                        t-out="billing_reference_vals.get('uuid')"/>
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
                <cbc:AdditionalAccountID t-out="accounting_supplier_vals.get('additional_account_id')"/>
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
            <cac:PaymentExchangeRate>
                <t t-set="exchange_rate_vals" t-value="vals.get('payment_exchange_rate_vals', {})"/>
                <cbc:SourceCurrencyCode t-out="exchange_rate_vals.get('source_currency_code')"/>
                <cbc:SourceCurrencyBaseRate t-out="exchange_rate_vals.get('source_currency_base_rate')"/>
                <cbc:TargetCurrencyCode t-out="exchange_rate_vals.get('target_currency_code')"/>
                <cbc:TargetCurrencyBaseRate t-out="exchange_rate_vals.get('target_currency_base_rate')"/>
                <cbc:CalculationRate t-out="exchange_rate_vals.get('calculation_rate')"/>
                <cbc:Date t-out="exchange_rate_vals.get('date')"/>
            </cac:PaymentExchangeRate>
            <cac:TaxTotal t-foreach="vals.get('tax_total_vals', [])" t-as="foreach_vals">
                <t t-call="{{TaxTotalType_template}}">
                    <t t-set="vals" t-value="foreach_vals"/>
                </t>
            </cac:TaxTotal>
        </t>
    </template>

    <template id="ubl_20_InvoiceType" inherit_id="account_edi_ubl_cii.ubl_20_CommonType" primary="True">
        <xpath expr="//*[local-name()='Note']" position="before">
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
                <cac:PrepaidPayment t-foreach="vals.get('prepaid_payments', [])" t-as="payment">
                    <cbc:ID t-out="payment.get('id')"/>
                    <cbc:PaidAmount
                        t-att="payment.get('paid_amount_attrs', {})"
                        t-out="format_float(payment.get('paid_amount'), vals.get('currency_dp'))"/>
                    <cbc:ReceivedDate t-out="payment.get('received_date')"/>
                    <cbc:PaidDate t-out="payment.get('paid_date')"/>
                    <cbc:InstructionID t-out="payment.get('instruction_id')"/>
                </cac:PrepaidPayment>
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
        <xpath expr="//*[local-name()='AccountingCustomerParty']" position="after">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
               xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cac:PrepaidPayment t-foreach="vals.get('prepaid_payments', [])" t-as="payment">
                    <cbc:ID t-out="payment.get('id')"/>
                    <cbc:PaidAmount
                        t-att="payment.get('paid_amount_attrs', {})"
                        t-out="format_float(payment.get('paid_amount'), vals.get('currency_dp'))"/>
                    <cbc:ReceivedDate t-out="payment.get('received_date')"/>
                    <cbc:PaidDate t-out="payment.get('paid_date')"/>
                    <cbc:InstructionID t-out="payment.get('instruction_id')"/>
                </cac:PrepaidPayment>
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
    <template id="ubl_21_AddressType" inherit_id="account_edi_ubl_cii.ubl_20_AddressType">
        <xpath expr="//*[local-name()='CountrySubentityCode']" position="after">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
               xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cac:AddressLine t-foreach="vals.get('address_lines', [])" t-as="address_line">
                    <cbc:Line t-out="address_line"/>
                </cac:AddressLine>
            </t>
        </xpath>
    </template>

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

    <template id="ubl_21_PartyType" inherit_id="account_edi_ubl_cii.ubl_20_PartyType">
        <xpath expr="//*[local-name()='EndpointID']" position="after">
            <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:IndustryClassificationCode
                        t-att="vals.get('industry_classification_code_attrs', {})"
                        t-out="vals.get('industry_classification_code')"/>
            </t>
        </xpath>
    </template>

    <template id="ubl_21_CreditNoteLineType" inherit_id="account_edi_ubl_cii.ubl_20_CreditNoteLineType" primary="True">
        <xpath expr="//*[local-name()='LineExtensionAmount']" position="after">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                <cac:InvoicePeriod t-foreach="vals.get('invoice_period_vals_list', [])" t-as="foreach_vals">
                    <t t-call="{{InvoicePeriodType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:InvoicePeriod>
            </t>
        </xpath>
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
        <xpath expr="//*[local-name()='ProfileID']" position="after">
            <cbc:ProfileExecutionID xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                t-out="vals.get('profile_execution_id')"/>
        </xpath>
        <xpath expr="//*[local-name()='IssueTime']" position="after">
            <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:DueDate t-out="vals.get('due_date')"/>
            </t>
        </xpath>
        <xpath expr="//*[local-name()='TaxCurrencyCode']" position="after">
            <t xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:BuyerReference t-out="vals.get('buyer_reference')"/>
            </t>
        </xpath>
        <xpath expr="//*[local-name()='LegalMonetaryTotal']" position="before">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                <cac:WithholdingTaxTotal t-foreach="vals.get('withholding_tax_total_vals_list', [])" t-as="foreach_vals">
                    <t t-call="{{TaxTotalType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:WithholdingTaxTotal>
            </t>
        </xpath>
    </template>

    <template id="ubl_21_InvoiceLineType" inherit_id="account_edi_ubl_cii.ubl_20_InvoiceLineType" primary="True">
        <xpath expr="//*[local-name()='TaxTotal']" position="after">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                <cac:WithholdingTaxTotal t-foreach="vals.get('withholding_tax_total_vals_list', [])" t-as="foreach_vals">
                    <t t-call="{{TaxTotalType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:WithholdingTaxTotal>
            </t>
        </xpath>
        <xpath expr="//*[local-name()='FreeOfChargeIndicator']" position="after">
            <t xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2">
                <cac:InvoicePeriod t-foreach="vals.get('invoice_period_vals_list', [])" t-as="foreach_vals">
                    <t t-call="{{InvoicePeriodType_template}}">
                        <t t-set="vals" t-value="foreach_vals"/>
                    </t>
                </cac:InvoicePeriod>
            </t>
        </xpath>
    </template>

    <template id="ubl_21_CreditNoteType" inherit_id="account_edi_ubl_cii.ubl_20_CreditNoteType" primary="True">
        <xpath expr="//*[local-name()='ProfileID']" position="after">
            <cbc:ProfileExecutionID xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                t-out="vals.get('profile_execution_id')"/>
        </xpath>
        <xpath expr="//*[local-name()='IssueTime']" position="after">
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
        <xpath expr="//*[local-name()='PaymentExchangeRate']" position="before">
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
        <xpath expr="//*[local-name()='ProfileID']" position="after">
            <cbc:ProfileExecutionID xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                t-out="vals.get('profile_execution_id')"/>
        </xpath>
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
from markupsafe import Markup

from odoo import _, models, Command
from odoo.addons.base.models.res_bank import sanitize_account_number
from odoo.exceptions import UserError, ValidationError
from odoo.tools import float_repr, format_list
from odoo.tools.float_utils import float_round
from odoo.tools.misc import clean_context, formatLang, html_escape
from odoo.tools.xml_utils import find_xml_value

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
    'BE': {'0208': 'company_registry', '9925': 'vat'},
    'BG': {'9926': 'vat'},
    'CH': {'9927': 'vat', '0183': None},
    'CY': {'9928': 'vat'},
    'CZ': {'9929': 'vat'},
    'DE': {'9930': 'vat'},
    'DK': {'0184': 'vat', '0198': 'vat'},
    'EE': {'9931': 'vat'},
    'ES': {'9920': 'vat'},
    'FI': {'0216': None, '0213': 'vat'},
    'FR': {'0009': 'siret', '9957': 'vat', '0002': None},
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
    'MY': {'0230': None},
    # Do not add the vat for NL, since: "[NL-R-003] For suppliers in the Netherlands, the legal entity identifier
    # MUST be either a KVK or OIN number (schemeID 0106 or 0190)" in the Bis 3 rules (in PartyLegalEntity/CompanyID).
    'NL': {'0106': None, '0190': None},
    'NO': {'0192': 'l10n_no_bronnoysund_number'},
    'NZ': {'0088': 'company_registry'},
    'PL': {'9945': 'vat'},
    'PT': {'9946': 'vat'},
    'RO': {'9947': 'vat'},
    'RS': {'9948': 'vat'},
    'SE': {'0007': 'company_registry', '9955': 'vat'},
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

    def _get_uom_unece_code(self, uom):
        """
        list of codes: https://docs.peppol.eu/poacc/billing/3.0/codelist/UNECERec20/
        or https://unece.org/fileadmin/DAM/cefact/recommendations/bkup_htm/add2c.htm (sorted by letter)
        """
        xmlid = uom.get_external_id()
        if xmlid and uom.id in xmlid:
            return UOM_TO_UNECE_CODE.get(xmlid[uom.id], 'C62')
        return 'C62'

    def _find_value(self, xpaths, tree, nsmap=False):
        """ Iteratively queries the tree using the xpaths and returns a result as soon as one is found """
        if not isinstance(xpaths, (tuple, list)):
            xpaths = [xpaths]
        for xpath in xpaths:
            # functions from ElementTree like "findtext" do not fully implement xpath, use "xpath" (from lxml) instead
            # (e.g. "//node[string-length(text()) > 5]" raises an invalidPredicate exception with "findtext")
            val = find_xml_value(xpath, tree, nsmap)
            if val:
                return val

    # -------------------------------------------------------------------------
    # TAXES
    # -------------------------------------------------------------------------

    def _validate_taxes(self, tax_ids):
        """ Validate the structure of the tax repartition lines (invalid structure could lead to unexpected results) """
        for tax in tax_ids:
            try:
                tax._validate_repartition_lines()
            except ValidationError as e:
                error_msg = _("Tax '%(tax_name)s' is invalid: %(error_message)s", tax_name=tax.name, error_message=e.args[0])  # args[0] gives the error message
                raise ValidationError(error_msg)

    def _get_tax_unece_codes(self, customer, supplier, tax):
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

    def _get_tax_category_list(self, customer, supplier, taxes):
        """ Full list: https://unece.org/fileadmin/DAM/trade/untdid/d16b/tred/tred5305.htm
        Subset: https://docs.peppol.eu/poacc/billing/3.0/codelist/UNCL5305/

        :param taxes:   account.tax records.
        :return:        A list of values to fill the TaxCategory foreach template.
        """
        res = []
        for tax in taxes:
            tax_unece_codes = self._get_tax_unece_codes(customer, supplier, tax)
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
            return custom_warning_message or _("The element %(record)s is required on %(field_list)s.", record=record, field_list=format_list(self.env, field_names))

        if not isinstance(field_names, (list, tuple)):
            field_names = (field_names,)

        has_values = any((field_name in record and record[field_name]) for field_name in field_names)
        # field is present
        if has_values:
            return

        # field is not present
        if custom_warning_message or isinstance(record, dict):
            return custom_warning_message or _(
                "The element %(record)s is required on %(field_list)s.",
                record=record,
                field_list=format_list(self.env, field_names),
            )

        display_field_names = record.fields_get(field_names)
        if len(field_names) == 1:
            display_field = f"'{display_field_names[field_names[0]]['string']}'"
            return _("The field %(field)s is required on %(record)s.", field=display_field, record=record.display_name)
        else:
            display_fields = format_list(self.env, [f"'{display_field_names[x]['string']}'" for x in display_field_names])
            return _("At least one of the following fields %(field_list)s is required on %(record)s.", field_list=display_fields, record=record.display_name)

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
            logs = self._import_fill_invoice(invoice, tree, qty_factor)

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

        attachments = self._import_attachments(invoice, tree)
        if attachments:
            invoice.with_context(no_new_invoice=True).message_post(attachment_ids=attachments.ids)

        return True

    def _import_attachments(self, invoice, tree):
        # Import the embedded PDF in the xml if some are found
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
                    invoice._message_set_main_attachment_id(attachment, force=True, filter_xml=False)
                attachments |= attachment

        return attachments

    def _import_partner(self, company_id, name, phone, email, vat, country_code=False, peppol_eas=False, peppol_endpoint=False, street=False, street2=False, city=False, zip_code=False):
        """ Retrieve the partner, if no matching partner is found, create it (only if he has a vat and a name) """
        logs = []
        if peppol_eas and peppol_endpoint:
            domain = [('peppol_eas', '=', peppol_eas), ('peppol_endpoint', '=', peppol_endpoint)]
        else:
            domain = False
        partner = self.env['res.partner'] \
            .with_company(company_id) \
            ._retrieve_partner(name=name, phone=phone, email=email, vat=vat, domain=domain)
        if not partner and name and vat:
            partner_vals = {'name': name, 'email': email, 'phone': phone, 'street': street, 'street2': street2, 'zip': zip_code, 'city': city}
            if peppol_eas and peppol_endpoint:
                partner_vals.update({'peppol_eas': peppol_eas, 'peppol_endpoint': peppol_endpoint})
            country = self.env.ref(f'base.{country_code.lower()}', raise_if_not_found=False) if country_code else False
            if country:
                partner_vals['country_id'] = country.id
            partner = self.env['res.partner'].create(partner_vals)
            if vat and self.env['res.partner']._run_vat_test(vat, country, partner.is_company):
                partner.vat = vat
            logs.append(_("Could not retrieve a partner corresponding to '%s'. A new partner was created.", name))
        return partner, logs

    def _import_partner_bank(self, invoice, bank_details):
        """ Retrieve the bank account, if no matching bank account is found, create it """
        # clear the context, because creation of partner when importing should not depend on the context default values
        ResPartnerBank = self.env['res.partner.bank'].with_env(self.env(context=clean_context(self.env.context)))
        bank_details = list(map(sanitize_account_number, bank_details))
        partner = self.env.company.partner_id if invoice.is_inbound() else invoice.partner_id
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

    def _import_document_allowance_charges(self, tree, record, tax_type, qty_factor=1):
        logs = []
        xpaths = self._get_document_allowance_charge_xpaths()
        line_vals = []
        for allow_el in tree.iterfind(xpaths['root']):
            name = allow_el.findtext(xpaths['reason']) or ""
            # Charge indicator factor: -1 for discount, 1 for charge
            charge_indicator = -1 if allow_el.findtext(xpaths['charge_indicator']).lower() == 'false' else 1
            amount = float(allow_el.findtext(xpaths['amount']) or 0)
            base_amount = float(allow_el.findtext(xpaths['base_amount']) or 0)
            if base_amount:
                price_unit = base_amount * charge_indicator * qty_factor
                percentage = float(allow_el.findtext(xpaths['percentage']) or 100)
                quantity = percentage / 100
            else:
                price_unit = amount * charge_indicator * qty_factor
                quantity = 1

            # Taxes
            tax_ids = []
            for tax_percent_node in allow_el.iterfind(xpaths['tax_percentage']):
                tax_amount = float(tax_percent_node.text)
                tax = self.env['account.tax'].search([
                    *self.env['account.tax']._check_company_domain(record.company_id),
                    ('amount', '=', tax_amount),
                    ('amount_type', '=', 'percent'),
                    ('type_tax_use', '=', tax_type),
                ], limit=1)
                if tax:
                    tax_ids += tax.ids
                elif name:
                    logs.append(_(
                        "Could not retrieve the tax: %(tax_percentage)s %% for line '%(line)s'.",
                        tax_percentage=tax_amount,
                        line=name,
                    ))
                else:
                    logs.append(
                        _("Could not retrieve the tax: %s for the document level allowance/charge.", tax_amount))

            line_vals.append([name, quantity, price_unit, tax_ids])
        return record._get_line_vals_list(line_vals), logs

    def _import_currency(self, tree, xpath):
        logs = []
        currency_name = tree.findtext(xpath)
        currency = self.env.company.currency_id
        if currency_name is not None:
            currency = currency.with_context(active_test=False).search([
                ('name', '=', currency_name),
            ], limit=1)
            if currency:
                if not currency.active:
                    logs.append(_("The currency '%s' is not active.", currency.name))
            else:
                logs.append(_("Could not retrieve currency: %s. Did you enable the multicurrency option "
                              "and activate the currency?", currency_name))
        return currency.id, logs

    def _import_description(self, tree, xpaths):
        description = ""
        for xpath in xpaths:
            note = tree.findtext(xpath)
            if note:
                description += f"<p>{html_escape(note)}</p>"
        return description

    def _import_prepaid_amount(self, invoice, tree, xpath, qty_factor):
        logs = []
        prepaid_amount = float(tree.findtext(xpath) or 0)
        if not invoice.currency_id.is_zero(prepaid_amount):
            amount = prepaid_amount * qty_factor
            formatted_amount = formatLang(self.env, amount, currency_obj=invoice.currency_id)
            logs.append(_("A payment of %s was detected.", formatted_amount))
        return logs

    def _import_invoice_lines(self, invoice, tree, xpath, qty_factor):
        logs = []
        lines_values = []
        for line_tree in tree.iterfind(xpath):
            line_values = self.with_company(invoice.company_id)._retrieve_invoice_line_vals(line_tree, invoice.move_type, qty_factor)
            line_values['tax_ids'], tax_logs = self._retrieve_taxes(
                invoice, line_values, invoice.journal_id.type,
            )
            logs += tax_logs
            if not line_values['product_uom_id']:
                line_values.pop('product_uom_id')  # if no uom, pop it so it's inferred from the product_id
            lines_values.append(line_values)
            lines_values += self._retrieve_line_charges(invoice, line_values, line_values['tax_ids'])
        return lines_values, logs

    def _retrieve_invoice_line_vals(self, tree, document_type=False, qty_factor=1):
        # Start and End date (enterprise fields)
        deferred_values = {}
        start_date = end_date = None
        if self.env['account.move.line']._fields.get('deferred_start_date'):
            start_date_node = tree.find('./{*}InvoicePeriod/{*}StartDate')
            end_date_node = tree.find('./{*}InvoicePeriod/{*}EndDate')
            if start_date_node is not None and end_date_node is not None:  # there is a constraint forcing none or the two to be set
                start_date = start_date_node.text
                end_date = end_date_node.text
            deferred_values = {
                'deferred_start_date': start_date,
                'deferred_end_date': end_date,
            }

        return {
            **self._retrieve_line_vals(tree, document_type, qty_factor),
            **deferred_values,
        }

    def _retrieve_line_vals(self, tree, document_type=False, qty_factor=1):
        """
        Read the xml invoice, extract the invoice line values, compute the odoo values
        to fill an invoice line form: quantity, price_unit, discount, product_uom_id.

        The way of computing invoice line is quite complicated:
        https://docs.peppol.eu/poacc/billing/3.0/bis/#_calculation_on_line_level (same as in factur-x documentation)

        line_net_subtotal = ( gross_unit_price - rebate ) * (delivered_qty / basis_qty) - allow_charge_amount

        with (UBL | CII):
            * net_unit_price = 'Price/PriceAmount' | 'NetPriceProductTradePrice' (mandatory) (BT-146)
            * gross_unit_price = 'Price/AllowanceCharge/BaseAmount' | 'GrossPriceProductTradePrice' (optional) (BT-148)
            * basis_qty = 'Price/BaseQuantity' | 'BasisQuantity' (optional, either below net_price node or
                gross_price node) (BT-149)
            * delivered_qty = 'InvoicedQuantity' (invoice) | 'BilledQuantity' (bill) | 'Quantity' (order) (mandatory) (BT-129)
            * allow_charge_amount = sum of 'AllowanceCharge' | 'SpecifiedTradeAllowanceCharge' (same level as Price)
                ON THE LINE level (optional) (BT-136 / BT-141)
            * line_net_subtotal = 'LineExtensionAmount' | 'LineTotalAmount' (mandatory) (BT-131)
            * rebate = 'Price/AllowanceCharge' | 'AppliedTradeAllowanceCharge' below gross_price node ! (BT-147)
                "item price discount" which is different from the usual allow_charge_amount
                gross_unit_price (BT-148) - rebate (BT-147) = net_unit_price (BT-146)

        In Odoo, we obtain:
        (1) = price_unit  =  gross_price_unit / basis_qty  =  (net_price_unit + rebate) / basis_qty
        (2) = quantity  =  delivered_qty
        (3) = discount (converted into a percentage)  =  100 * (1 - price_subtotal / (delivered_qty * price_unit))
        (4) = price_subtotal

        Alternatively, we could also set: quantity = delivered_qty/basis_qty

        WARNING, the basis quantity parameter is annoying, for instance, an invoice with a line:
            item A  | price per unit of measure/unit price: 30  | uom = 3 pieces | billed qty = 3 | rebate = 2  | untaxed total = 28
        Indeed, 30 $ / 3 pieces = 10 $ / piece => 10 * 3 (billed quantity) - 2 (rebate) = 28

        UBL ROUNDING: "the result of Item line net
            amount = ((Item net price (BT-146)÷Item price base quantity (BT-149))×(Invoiced Quantity (BT-129))
        must be rounded to two decimals, and the allowance/charge amounts are also rounded separately."
        It is not possible to do it in Odoo.
        """
        xpath_dict = self._get_line_xpaths(document_type, qty_factor)
        # basis_qty (optional)
        basis_qty = float(self._find_value(xpath_dict['basis_qty'], tree) or 1)

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

        # delivered_qty (mandatory)
        delivered_qty = 1
        product_vals = {k: self._find_value(v, tree) for k, v in xpath_dict['product'].items()}
        product = self._import_product(**product_vals)
        product_uom = self.env['uom.uom']
        quantity_node = tree.find(xpath_dict['delivered_qty'])
        if quantity_node is not None:
            delivered_qty = float(quantity_node.text)
            uom_xml = quantity_node.attrib.get('unitCode')
            if uom_xml:
                uom_infered_xmlid = [
                    odoo_xmlid for odoo_xmlid, uom_unece in UOM_TO_UNECE_CODE.items() if uom_unece == uom_xml
                ]
                if uom_infered_xmlid:
                    product_uom = self.env.ref(uom_infered_xmlid[0], raise_if_not_found=False) or self.env['uom.uom']
        if product and product_uom and product_uom.category_id != product.product_tmpl_id.uom_id.category_id:
            # uom incompatibility
            product_uom = self.env['uom.uom']

        # line_net_subtotal (mandatory)
        price_subtotal = None
        line_total_amount_node = tree.find(xpath_dict['line_total_amount'])
        if line_total_amount_node is not None:
            price_subtotal = float(line_total_amount_node.text)

        # quantity
        quantity = delivered_qty * qty_factor

        # Charges are collected (they are used to create new lines), Allowances are transformed into discounts
        charges = []
        discount_amount = 0
        for allowance_charge_node in tree.iterfind(xpath_dict['allowance_charge']):
            charge_indicator = allowance_charge_node.findtext(xpath_dict['allowance_charge_indicator'])
            amount = float(allowance_charge_node.findtext(xpath_dict['allowance_charge_amount'], default='0'))
            reason_code = allowance_charge_node.findtext(xpath_dict['allowance_charge_reason_code'], default='')
            reason = allowance_charge_node.findtext(xpath_dict['allowance_charge_reason'], default='')
            if charge_indicator.lower() == 'true':
                charges.append({
                    'amount': amount,
                    'line_quantity': quantity,
                    'reason': reason,
                    'reason_code': reason_code,
                })
            else:
                discount_amount += amount

        # price_unit
        charge_amount = sum(d['amount'] for d in charges)
        allow_charge_amount = discount_amount - charge_amount
        if gross_price_unit is not None:
            price_unit = gross_price_unit / basis_qty
        elif net_price_unit is not None:
            price_unit = (net_price_unit + rebate) / basis_qty
        elif price_subtotal is not None:
            price_unit = (price_subtotal + allow_charge_amount) / (delivered_qty or 1)
        else:
            raise UserError(_("No gross price, net price nor line subtotal amount found for line in xml"))

        # discount
        discount = 0
        if delivered_qty * price_unit != 0 and price_subtotal is not None:
            discount = 100 * (1 - (price_subtotal - charge_amount) / (delivered_qty * price_unit))

        # Sometimes, the xml received is very bad; e.g.:
        #   * unit price = 0, qty = 0, but price_subtotal = -200
        #   * unit price = 0, qty = 1, but price_subtotal = -200
        #   * unit price = 1, qty = 0, but price_subtotal = -200
        # for instance, when filling a down payment as an document line. The equation in the docstring is not
        # respected, and the result will not be correct, so we just follow the simple rule below:
        if net_price_unit is not None and price_subtotal != net_price_unit * (delivered_qty / basis_qty) - allow_charge_amount:
            if net_price_unit == 0 and delivered_qty == 0:
                quantity = 1
                price_unit = price_subtotal
            elif net_price_unit == 0:
                price_unit = price_subtotal / delivered_qty
            elif delivered_qty == 0:
                quantity = price_subtotal / price_unit

        return {
            # vals to be written on the document line
            'name': self._find_value(xpath_dict['name'], tree),
            'product_id': product.id,
            'product_uom_id': product_uom.id,
            'price_unit': price_unit,
            'quantity': quantity,
            'discount': discount,
            'tax_nodes': self._get_tax_nodes(tree),  # see `_retrieve_taxes`
            'charges': charges,  # see `_retrieve_line_charges`
        }

    def _import_product(self, **product_vals):
        return self.env['product.product']._retrieve_product(**product_vals)

    def _retrieve_fixed_tax(self, company_id, fixed_tax_vals):
        """ Retrieve the fixed tax at import, iteratively search for a tax:
        1. not price_include matching the name and the amount
        2. not price_include matching the amount
        3. price_include matching the name and the amount
        4. price_include matching the amount
        """
        base_domain = [
            *self.env['account.journal']._check_company_domain(company_id),
            ('amount_type', '=', 'fixed'),
            ('amount', '=', fixed_tax_vals['amount']),
        ]
        for price_include in (False, True):
            for name in (fixed_tax_vals['reason'], False):
                domain = base_domain + [('price_include', '=', price_include)]
                if name:
                    domain.append(('name', '=', name))
                tax = self.env['account.tax'].search(domain, limit=1)
                if tax:
                    return tax
        return self.env['account.tax']

    def _retrieve_taxes(self, record, line_values, tax_type):
        """
        Retrieve the taxes on the document line at import.

        In a UBL/CII xml, the Odoo "price_include" concept does not exist. Hence, first look for a price_include=False,
        if it is unsuccessful, look for a price_include=True.
        """
        # Taxes: all amounts are tax excluded, so first try to fetch price_include=False taxes,
        # if no results, try to fetch the price_include=True taxes. If results, need to adapt the price_unit.
        logs = []
        taxes = []
        for tax_node in line_values.pop('tax_nodes'):
            amount = float(tax_node.text)
            domain = [
                *self.env['account.journal']._check_company_domain(record.company_id),
                ('amount_type', '=', 'percent'),
                ('type_tax_use', '=', tax_type),
                ('amount', '=', amount),
            ]
            tax = self.env['account.tax']
            if hasattr(record, '_get_specific_tax'):
                tax = record._get_specific_tax(line_values['name'], 'percent', amount, tax_type)
            if not tax:
                tax = self.env['account.tax'].search(domain + [('price_include', '=', False)], limit=1)
            if not tax:
                tax = self.env['account.tax'].search(domain + [('price_include', '=', True)], limit=1)

            if not tax:
                logs.append(
                    _("Could not retrieve the tax: %(amount)s %% for line '%(line)s'.",
                    amount=amount,
                    line=line_values['name']),
                )
            else:
                taxes.append(tax.id)
                if tax.price_include:
                    line_values['price_unit'] *= (1 + tax.amount / 100)
        return taxes, logs

    def _retrieve_line_charges(self, record, line_values, taxes):
        """
        Handle the charges on the document line at import.

        For each charge on the line, it creates a new aml.
        Special case: if the ReasonCode == 'AEO', there is a high chance the xml was produced by Odoo and the
        corresponding line had a fixed tax, so it first tries to find a matching fixed tax to apply to the current aml.
        """
        charges_vals = []
        for charge in line_values.pop('charges'):
            if charge['reason_code'] == 'AEO':
                # a 1 eur fixed tax on a line with quantity=2 will yield an AllowanceCharge with amount = 2
                charge_copy = charge.copy()
                charge_copy['amount'] /= charge_copy['line_quantity']
                if tax := self._retrieve_fixed_tax(record.company_id, charge_copy):
                    taxes.append(tax.id)
                    if tax.price_include:
                        line_values['price_unit'] += tax.amount
                    continue
            charges_vals.append([
                charge['reason_code'] + " " + charge['reason'],
                1,
                charge['amount'],
                taxes,
            ])
        return record._get_line_vals_list(charges_vals)

    def _get_document_allowance_charge_xpaths(self):
        # OVERRIDE
        pass

    def _get_invoice_line_xpaths(self, invoice_line, qty_factor):
        # OVERRIDE
        pass

    def _correct_invoice_tax_amount(self, tree, invoice):
        pass  # To be implemented by the format if needed

```

## File: models\account_edi_xml_cii_facturx.py

```python
from odoo import _, models, Command
from odoo.tools import float_repr, is_html_empty, html2plaintext, cleanup_xml_node
from lxml import etree

from datetime import datetime

import logging

_logger = logging.getLogger(__name__)

DEFAULT_FACTURX_DATE_FORMAT = '%Y%m%d'
CII_NAMESPACES = {
    'ram': "urn:un:unece:uncefact:data:standard:ReusableAggregateBusinessInformationEntity:100",
    'rsm': "urn:un:unece:uncefact:data:standard:CrossIndustryInvoice:100",
    'udt': "urn:un:unece:uncefact:data:standard:UnqualifiedDataType:100",
}


class AccountEdiXmlCII(models.AbstractModel):
    _name = "account.edi.xml.cii"
    _inherit = 'account.edi.common'
    _description = "Factur-x/XRechnung CII 2.2.0"

    def _find_value(self, xpath, tree, nsmap=False):
        # EXTENDS account.edi.common
        return super()._find_value(xpath, tree, CII_NAMESPACES)

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

        def grouping_key_generator(base_line, tax_data):
            tax = tax_data['tax']
            customer = invoice.commercial_partner_id
            supplier = invoice.company_id.partner_id.commercial_partner_id
            grouping_key = {
                **self._get_tax_unece_codes(customer, supplier, tax),
                'amount': tax.amount,
                'amount_type': tax.amount_type,
            }
            # If the tax is fixed, we want to have one group per tax
            # s.t. when the invoice is imported, we can try to guess the fixed taxes
            if tax.amount_type == 'fixed':
                grouping_key['tax_name'] = tax.name
            return grouping_key

        # Validate the structure of the taxes
        self._validate_taxes(invoice.invoice_line_ids.tax_ids)

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
            line_vals['unece_uom_code'] = self._get_uom_unece_code(line.product_uom_id)

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

            # The quantity is the line.quantity since we keep the unece_uom_code!
            line_vals['quantity'] = line_vals['line'].quantity

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

    def _import_retrieve_partner_vals(self, tree, role):
        return {
            'vat': self._find_value(f".//ram:{role}/ram:SpecifiedTaxRegistration/ram:ID[string-length(text()) > 5]", tree),
            'name': self._find_value(f".//ram:{role}/ram:Name", tree),
            'phone': self._find_value(f".//ram:{role}/ram:DefinedTradeContact/ram:TelephoneUniversalCommunication/ram:CompleteNumber", tree),
            'email': self._find_value(f".//ram:{role}//ram:EmailURIUniversalCommunication/ram:URIID", tree),
            'country_code': self._find_value(f'.//ram:{role}/ram:PostalTradeAddress//ram:CountryID', tree),
        }

    def _import_fill_invoice(self, invoice, tree, qty_factor):
        logs = []
        invoice_values = {}
        if qty_factor == -1:
            logs.append(_("The invoice has been converted into a credit note and the quantities have been reverted."))
        role = 'SellerTradeParty' if invoice.journal_id.type == 'purchase' else 'BuyerTradeParty'
        partner, partner_logs = self._import_partner(invoice.company_id, **self._import_retrieve_partner_vals(tree, role))
        # Need to set partner before to compute bank and lines properly
        invoice.partner_id = partner.id
        invoice_values['currency_id'], currency_logs = self._import_currency(tree, './/{*}InvoiceCurrencyCode')

        # ==== partner_bank_id ====
        bank_detail_nodes = tree.findall('.//{*}SpecifiedTradeSettlementPaymentMeans')
        bank_details = [
            bank_detail_node.findtext('{*}PayeePartyCreditorFinancialAccount/{*}IBANID')
            or bank_detail_node.findtext('{*}PayeePartyCreditorFinancialAccount/{*}ProprietaryID')
            for bank_detail_node in bank_detail_nodes
        ]
        if bank_details:
            self._import_partner_bank(invoice, bank_details=bank_details)

        # ==== ref, invoice_origin, narration, payment_reference ====
        invoice_values['ref'] = tree.findtext('./{*}ExchangedDocument/{*}ID')
        invoice_values['invoice_origin'] = tree.findtext(
            './/{*}BuyerOrderReferencedDocument/{*}IssuerAssignedID'
        )
        invoice_values['narration'] = self._import_description(tree, xpaths=[
            './{*}ExchangedDocument/{*}IncludedNote/{*}Content',
            './/{*}SpecifiedTradePaymentTerms/{*}Description',
        ])
        invoice_values['payment_reference'] = tree.findtext(
            './{*}SupplyChainTradeTransaction/{*}ApplicableHeaderTradeSettlement/{*}PaymentReference'
        )

        # ==== invoice_date, invoice_date_due ====
        issue_date = tree.findtext('./{*}ExchangedDocument/{*}IssueDateTime/{*}DateTimeString')
        if issue_date:
            invoice_values['invoice_date'] = datetime.strptime(issue_date.strip(), DEFAULT_FACTURX_DATE_FORMAT)
        due_date = tree.findtext('.//{*}SpecifiedTradePaymentTerms/{*}DueDateDateTime/{*}DateTimeString')
        if due_date:
            invoice_values['invoice_date_due'] = datetime.strptime(due_date.strip(), DEFAULT_FACTURX_DATE_FORMAT)

        # ==== Document level AllowanceCharge, Prepaid Amounts, Invoice Lines ====
        allowance_charges_line_vals, allowance_charges_logs = self._import_document_allowance_charges(
            tree, invoice, invoice.journal_id.type, qty_factor,
        )
        logs += self._import_prepaid_amount(invoice, tree, './/{*}ApplicableHeaderTradeSettlement/{*}SpecifiedTradeSettlementHeaderMonetarySummation/{*}TotalPrepaidAmount', qty_factor)
        invoice_line_vals, line_logs = self._import_invoice_lines(invoice, tree, './{*}SupplyChainTradeTransaction/{*}IncludedSupplyChainTradeLineItem', qty_factor)
        line_vals = allowance_charges_line_vals + invoice_line_vals

        invoice_values = {
            **invoice_values,
            'invoice_line_ids': [Command.create(line_value) for line_value in line_vals],
        }
        invoice.write(invoice_values)
        logs += partner_logs + currency_logs + line_logs + allowance_charges_logs
        return logs

    def _get_tax_nodes(self, tree):
        return tree.findall('.//{*}ApplicableTradeTax/{*}RateApplicablePercent')

    def _get_document_allowance_charge_xpaths(self):
        return {
            'root': './{*}SupplyChainTradeTransaction/{*}ApplicableHeaderTradeSettlement/{*}SpecifiedTradeAllowanceCharge',
            'charge_indicator': './{*}ChargeIndicator/{*}Indicator',
            'base_amount': './{*}BasisAmount',
            'amount': './{*}ActualAmount',
            'reason': './{*}Reason',
            'percentage': './{*}CalculationPercent',
            'tax_percentage': './{*}CategoryTradeTax/{*}RateApplicablePercent',
        }

    def _get_line_xpaths(self, document_type=False, qty_factor=1):
        return {
            'basis_qty': (
                './ram:SpecifiedLineTradeAgreement/ram:GrossPriceProductTradePrice/ram:BasisQuantity',
                './ram:SpecifiedLineTradeAgreement/ram:NetPriceProductTradePrice/ram:BasisQuantity',
            ),
            'gross_price_unit': './{*}SpecifiedLineTradeAgreement/{*}GrossPriceProductTradePrice/{*}ChargeAmount',
            'rebate': './{*}SpecifiedLineTradeAgreement/{*}GrossPriceProductTradePrice/{*}AppliedTradeAllowanceCharge/{*}ActualAmount',
            'net_price_unit': './{*}SpecifiedLineTradeAgreement/{*}NetPriceProductTradePrice/{*}ChargeAmount',
            'delivered_qty': './{*}SpecifiedLineTradeDelivery/{*}BilledQuantity',
            'allowance_charge': './/{*}SpecifiedLineTradeSettlement/{*}SpecifiedTradeAllowanceCharge',
            'allowance_charge_indicator': './{*}ChargeIndicator/{*}Indicator',
            'allowance_charge_amount': './{*}ActualAmount',
            'allowance_charge_reason': './{*}Reason',
            'allowance_charge_reason_code': './{*}ReasonCode',
            'line_total_amount': './{*}SpecifiedLineTradeSettlement/{*}SpecifiedTradeSettlementLineMonetarySummation/{*}LineTotalAmount',
            'name': [
                './ram:SpecifiedTradeProduct/ram:Name',
            ],
            'product': {
                'default_code': './ram:SpecifiedTradeProduct/ram:SellerAssignedID',
                'name': './ram:SpecifiedTradeProduct/ram:Name',
                'barcode': './ram:SpecifiedTradeProduct/ram:GlobalID',
            },
        }

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
from collections import defaultdict
from lxml import etree

from odoo import _, models, Command
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
        if invoice.move_type == 'out_invoice':
            if invoice.partner_bank_id:
                payment_means_code, payment_means_name = (30, 'credit transfer')
            else:
                payment_means_code, payment_means_name = ('ZZZ', 'mutually defined')
        else:
            payment_means_code, payment_means_name = (57, 'standing agreement')

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
            if grouping_key['tax_amount_type'] != 'fixed' or not self._context.get('convert_fixed_taxes'):
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
        taxes = line.tax_ids.flatten_taxes_hierarchy()
        if self._context.get('convert_fixed_taxes'):
            taxes = taxes.filtered(lambda t: t.amount_type != 'fixed')
        customer = line.move_id.commercial_partner_id
        supplier = line.move_id.company_id.partner_id.commercial_partner_id
        tax_category_vals_list = self._get_tax_category_list(customer, supplier, taxes)
        description = line.name and line.name.replace('\n', ' ')
        return {
            'description': description,
            'name': product.name or description,
            'sellers_item_identification_vals': {'id': product.code},
            'classified_tax_category_vals': tax_category_vals_list,
        }

    def _get_document_allowance_charge_vals_list(self, invoice, taxes_vals=None):
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
        if self._context.get('convert_fixed_taxes'):
            for grouping_key, tax_details in tax_values_list['tax_details'].items():
                if grouping_key['tax_amount_type'] == 'fixed':
                    fixed_tax_charge_vals_list.append({
                        'currency_name': line.currency_id.name,
                        'currency_dp': self._get_currency_decimal_places(line.currency_id),
                        'charge_indicator': 'true',
                        'allowance_charge_reason_code': 'AEO',
                        'allowance_charge_reason': grouping_key['tax_name'],
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

        uom = self._get_uom_unece_code(line.product_uom_id)

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

        uom = self._get_uom_unece_code(line.product_uom_id)
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

    def _get_tax_grouping_key(self, base_line, tax_data):
        tax = tax_data['tax']
        customer = base_line['record'].move_id.commercial_partner_id
        supplier = base_line['record'].move_id.company_id.partner_id.commercial_partner_id
        tax_category_vals = self._get_tax_category_list(customer, supplier, tax)[0]
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

    def _export_invoice_vals(self, invoice):
        # Validate the structure of the taxes
        self._validate_taxes(invoice.invoice_line_ids.tax_ids)

        # Compute the tax details for the whole invoice and each invoice line separately.
        taxes_vals = invoice._prepare_invoice_aggregated_taxes(
            grouping_key_generator=self._get_tax_grouping_key,
            filter_tax_values_to_apply=self._apply_invoice_tax_filter,
            filter_invl_to_apply=self._apply_invoice_line_filter,
            round_from_tax_lines=True,
        )

        # Fixed Taxes: filter them on the document level, and adapt the totals
        # Fixed taxes are not supposed to be taxes in real live. However, this is the way in Odoo to manage recupel
        # taxes in Belgium. Since only one tax is allowed, the fixed tax is removed from totals of lines but added
        # as an extra charge/allowance.
        if self._context.get('convert_fixed_taxes'):
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
        document_allowance_charge_vals_list = self._get_document_allowance_charge_vals_list(invoice, taxes_vals)
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
            'InvoicePeriodType_template': 'account_edi_ubl_cii.ubl_20_InvoicePeriodType',
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
                'document_currency_code': invoice.currency_id.name,
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

    def _export_invoice(self, invoice, convert_fixed_taxes=True):
        """ Generates an UBL 2.0 xml for a given invoice.
        :param convert_fixed_taxes: whether the fixed taxes are converted into AllowanceCharges on the InvoiceLines
        """
        vals = self \
            .with_context(convert_fixed_taxes=convert_fixed_taxes) \
            ._export_invoice_vals(invoice.with_context(lang=invoice.partner_id.lang))
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
            'vat': self._find_value(f'.//cac:{role}Party/cac:Party//cbc:CompanyID[string-length(text()) > 5]', tree),
            'phone': self._find_value(f'.//cac:{role}Party/cac:Party//cbc:Telephone', tree),
            'email': self._find_value(f'.//cac:{role}Party/cac:Party//cbc:ElectronicMail', tree),
            'name': self._find_value(f'.//cac:{role}Party/cac:Party//cbc:Name', tree) or
                    self._find_value(f'.//cac:{role}Party/cac:Party//cbc:RegistrationName', tree),
            'country_code': self._find_value(f'.//cac:{role}Party/cac:Party//cac:Country//cbc:IdentificationCode', tree),
            'street': self._find_value(f'.//cac:{role}Party/cac:Party//cbc:StreetName', tree),
            'street2': self._find_value(f'.//cac:{role}Party/cac:Party//cbc:AdditionalStreetName', tree),
            'city': self._find_value(f'.//cac:{role}Party/cac:Party//cbc:CityName', tree),
            'zip_code': self._find_value(f'.//cac:{role}Party/cac:Party//cbc:PostalZone', tree),
        }

    def _import_fill_invoice(self, invoice, tree, qty_factor):
        logs = []
        invoice_values = {}
        if qty_factor == -1:
            logs.append(_("The invoice has been converted into a credit note and the quantities have been reverted."))
        role = "AccountingCustomer" if invoice.journal_id.type == 'sale' else "AccountingSupplier"
        partner, partner_logs = self._import_partner(invoice.company_id, **self._import_retrieve_partner_vals(tree, role))
        # Need to set partner before to compute bank and lines properly
        invoice.partner_id = partner.id
        invoice_values['currency_id'], currency_logs = self._import_currency(tree, './/{*}DocumentCurrencyCode')
        invoice_values['invoice_date'] = tree.findtext('./{*}IssueDate')
        invoice_values['invoice_date_due'] = self._find_value(('./cbc:DueDate', './/cbc:PaymentDueDate'), tree)
        # ==== partner_bank_id ====
        bank_detail_nodes = tree.findall('.//{*}PaymentMeans')
        bank_details = [bank_detail_node.findtext('{*}PayeeFinancialAccount/{*}ID') for bank_detail_node in bank_detail_nodes]
        if bank_details:
            self._import_partner_bank(invoice, bank_details)

        # ==== ref, invoice_origin, narration, payment_reference ====
        ref = tree.findtext('./{*}ID')
        if ref and invoice.is_sale_document(include_receipts=True) and invoice.quick_edit_mode:
            invoice_values['name'] = ref
        elif ref:
            invoice_values['ref'] = ref
        invoice_values['invoice_origin'] = tree.findtext('./{*}OrderReference/{*}ID')
        invoice_values['narration'] = self._import_description(tree, xpaths=['./{*}Note', './{*}PaymentTerms/{*}Note'])
        invoice_values['payment_reference'] = tree.findtext('./{*}PaymentMeans/{*}PaymentID')

        # ==== Delivery ====
        delivery_date = tree.find('.//{*}Delivery/{*}ActualDeliveryDate')
        invoice.delivery_date = delivery_date is not None and delivery_date.text

        # ==== invoice_incoterm_id ====
        incoterm_code = tree.findtext('./{*}TransportExecutionTerms/{*}DeliveryTerms/{*}ID')
        if incoterm_code:
            incoterm = self.env['account.incoterms'].search([('code', '=', incoterm_code)], limit=1)
            if incoterm:
                invoice_values['invoice_incoterm_id'] = incoterm.id

        # ==== Document level AllowanceCharge, Prepaid Amounts, Invoice Lines ====
        allowance_charges_line_vals, allowance_charges_logs = self._import_document_allowance_charges(tree, invoice, invoice.journal_id.type, qty_factor)
        logs += self._import_prepaid_amount(invoice, tree, './{*}LegalMonetaryTotal/{*}PrepaidAmount', qty_factor)
        line_tag = (
            'InvoiceLine'
            if invoice.move_type in ('in_invoice', 'out_invoice') or qty_factor == -1
            else 'CreditNoteLine'
        )
        invoice_line_vals, line_logs = self._import_invoice_lines(invoice, tree, './{*}' + line_tag, qty_factor)
        line_vals = allowance_charges_line_vals + invoice_line_vals

        invoice_values = {
            **invoice_values,
            'invoice_line_ids': [Command.create(line_value) for line_value in line_vals],
        }
        invoice.write(invoice_values)
        logs += partner_logs + currency_logs + line_logs + allowance_charges_logs
        return logs

    def _get_tax_nodes(self, tree):
        tax_nodes = tree.findall('.//{*}Item/{*}ClassifiedTaxCategory/{*}Percent')
        if not tax_nodes:
            for elem in tree.findall('.//{*}TaxTotal'):
                percentage_nodes = elem.findall('.//{*}TaxSubtotal/{*}TaxCategory/{*}Percent')
                if not percentage_nodes:
                    percentage_nodes = elem.findall('.//{*}TaxSubtotal/{*}Percent')
                tax_nodes += percentage_nodes
        return tax_nodes

    def _get_document_allowance_charge_xpaths(self):
        return {
            'root': './{*}AllowanceCharge',
            'charge_indicator': './{*}ChargeIndicator',
            'base_amount': './{*}BaseAmount',
            'amount': './{*}Amount',
            'reason': './{*}AllowanceChargeReason',
            'percentage': './{*}MultiplierFactorNumeric',
            'tax_percentage': './{*}TaxCategory/{*}Percent',
        }

    def _get_line_xpaths(self, document_type=False, qty_factor=1):
        return {
            'basis_qty': './cac:Price/cbc:BaseQuantity',
            'gross_price_unit': './{*}Price/{*}AllowanceCharge/{*}BaseAmount',
            'rebate': './{*}Price/{*}AllowanceCharge/{*}Amount',
            'net_price_unit': './{*}Price/{*}PriceAmount',
            'delivered_qty': (
                './{*}InvoicedQuantity'
                if document_type and document_type in ('in_invoice', 'out_invoice') or qty_factor == -1
                else './{*}CreditedQuantity'
            ),
            'allowance_charge': './/{*}AllowanceCharge',
            'allowance_charge_indicator': './{*}ChargeIndicator',
            'allowance_charge_amount': './{*}Amount',
            'allowance_charge_reason': './{*}AllowanceChargeReason',
            'allowance_charge_reason_code': './{*}AllowanceChargeReasonCode',
            'line_total_amount': './{*}LineExtensionAmount',
            'name': [
                './cac:Item/cbc:Description',
                './cac:Item/cbc:Name',
            ],
            'product': {
                'default_code': './cac:Item/cac:SellersItemIdentification/cbc:ID',
                'name': './cac:Item/cbc:Name',
                'barcode': './cac:Item/cac:StandardItemIdentification/cbc:ID[@schemeID="0160"]',
            },
        }

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
            'AddressType_template': 'account_edi_ubl_cii.ubl_21_AddressType',
            'PaymentTermsType_template': 'account_edi_ubl_cii.ubl_21_PaymentTermsType',
            'PartyType_template': 'account_edi_ubl_cii.ubl_21_PartyType',
            'InvoiceLineType_template': 'account_edi_ubl_cii.ubl_21_InvoiceLineType',
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

    def _get_tax_category_list(self, customer, supplier, taxes):
        # EXTENDS account.edi.xml.ubl_bis3
        vals_list = super()._get_tax_category_list(customer, supplier, taxes)
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

    def _get_tax_category_list(self, customer, supplier, taxes):
        # EXTENDS account.edi.xml.ubl_21
        vals_list = super()._get_tax_category_list(customer, supplier, taxes)

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
        endpoint_node = tree.find(f'.//cac:{role}Party/cac:Party/cbc:EndpointID', UBL_NAMESPACES)
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

    def _get_tax_category_list(self, customer, supplier, taxes):
        # EXTENDS account.edi.xml.ubl_bis3
        vals_list = super()._get_tax_category_list(customer, supplier, taxes)
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

    def _get_tax_sg_codes(self, tax):
        """ https://www.peppolguide.sg/billing/bis/#_gst_category_codes
        """
        tax_category_code = 'SR'
        if tax.amount == 0:
            tax_category_code = 'ZR'
        return tax_category_code

    def _get_tax_category_list(self, customer, supplier, taxes):
        # OVERRIDE
        res = []
        for tax in taxes:
            res.append({
                'id': self._get_tax_sg_codes(tax),
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
from odoo import _, api, fields, models, Command


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
    # ACTIONS
    # -------------------------------------------------------------------------

    def action_invoice_download_ubl(self):
        if invoices_with_ubl := self.filtered('ubl_cii_xml_id'):
            return {
                'type': 'ir.actions.act_url',
                'url': f'/account/download_invoice_documents/{",".join(map(str, invoices_with_ubl.ids))}/ubl',
                'target': 'download',
            }
        return False

    # -------------------------------------------------------------------------
    # BUSINESS
    # -------------------------------------------------------------------------

    def _get_fields_to_detach(self):
        # EXTENDS account
        fields_list = super()._get_fields_to_detach()
        fields_list.append("ubl_cii_xml_file")
        return fields_list

    def _get_invoice_legal_documents(self, filetype, allow_fallback=False):
        # EXTENDS account
        if filetype == 'ubl':
            if ubl_attachment := self.ubl_cii_xml_id:
                return {
                    'filename': ubl_attachment.name,
                    'filetype': 'xml',
                    'content': ubl_attachment.raw,
                }
        return super()._get_invoice_legal_documents(filetype, allow_fallback=allow_fallback)

    def get_extra_print_items(self):
        print_items = super().get_extra_print_items()
        if self.ubl_cii_xml_id:
            print_items.append({
                'key': 'download_ubl',
                'description': _('XML UBL'),
                **self.action_invoice_download_ubl(),
            })
        return print_items

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

    def _need_ubl_cii_xml(self, ubl_cii_format):
        self.ensure_one()
        return not self.ubl_cii_xml_id \
            and self.is_sale_document() \
            and ubl_cii_format in self.env['res.partner']._get_ubl_cii_formats()

    @api.model
    def _get_line_vals_list(self, lines_vals):
        """ Get invoice line values list.

        param list line_vals: List of values [name, qty, price, tax].
        :return: List of invoice line values.
        """
        return [{
            'sequence': 0,  # be sure to put these lines above the 'real' invoice lines
            'name': name,
            'quantity': quantity,
            'price_unit': price_unit,
            'tax_ids': [Command.set(tax_ids)],
        } for name, quantity, price_unit, tax_ids in lines_vals]

    def _get_specific_tax(self, name, amount_type, amount, tax_type):
        AccountMoveLine = self.env['account.move.line']
        if hasattr(AccountMoveLine, '_predict_specific_tax'):
            # company check is already done in the prediction query
            predicted_tax_id = AccountMoveLine._predict_specific_tax(
                self, name, self.partner_id, amount_type, amount, tax_type,
            )
            return self.env['account.tax'].browse(predicted_tax_id)
        return self.env['account.tax']

```

## File: models\account_move_send.py

```python
import base64
import logging
import io

from lxml import etree
from xml.sax.saxutils import escape, quoteattr

from odoo import _, api, fields, models, tools, SUPERUSER_ID
from odoo.tools import cleanup_xml_node
from odoo.tools.pdf import OdooPdfFileReader, OdooPdfFileWriter

_logger = logging.getLogger(__name__)


class AccountMoveSend(models.AbstractModel):
    _inherit = 'account.move.send'

    # -------------------------------------------------------------------------
    # ALERTS
    # -------------------------------------------------------------------------

    def _get_alerts(self, moves, moves_data):
        # EXTENDS 'account'
        alerts = super()._get_alerts(moves, moves_data)

        peppol_formats = set(self.env['res.partner']._get_peppol_formats())
        if peppol_format_moves := moves.filtered(lambda m: moves_data[m]['invoice_edi_format'] in peppol_formats):
            not_configured_company_partners = peppol_format_moves.company_id.partner_id.filtered(
                lambda partner: not (partner.peppol_eas and partner.peppol_endpoint)
            )
            if not_configured_company_partners:
                alerts['account_edi_ubl_cii_configure_company'] = {
                    'message': _("Please fill in your company's VAT or Peppol Address to generate a complete XML file."),
                    'level': 'info',
                    'action_text': _("Configure"),
                    'action': not_configured_company_partners._get_records_action(),
                }
            not_configured_partners = peppol_format_moves.partner_id.commercial_partner_id.filtered(
                lambda partner: not (partner.peppol_eas and partner.peppol_endpoint)
            )
            if not_configured_partners:
                alerts['account_edi_ubl_cii_configure_partner'] = {
                    'message': _("Please fill in partner's VAT or Peppol Address."),
                    'level': 'info',
                    'action_text': _("View Partner(s)"),
                    'action': not_configured_partners._get_records_action(name=_("Check Partner(s)"))
                }
        return alerts

    # -------------------------------------------------------------------------
    # ATTACHMENTS
    # -------------------------------------------------------------------------

    def _get_invoice_extra_attachments(self, move):
        # EXTENDS 'account'
        return super()._get_invoice_extra_attachments(move) + move.ubl_cii_xml_id

    def _get_placeholder_mail_attachments_data(self, move, invoice_edi_format=None, extra_edis=None):
        # EXTENDS 'account'
        results = super()._get_placeholder_mail_attachments_data(move, invoice_edi_format=invoice_edi_format, extra_edis=extra_edis)
        if move._need_ubl_cii_xml(invoice_edi_format):
            builder = move.partner_id.commercial_partner_id._get_edi_builder(invoice_edi_format)
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

    def _hook_invoice_document_before_pdf_report_render(self, invoice, invoice_data):
        # EXTENDS 'account'
        super()._hook_invoice_document_before_pdf_report_render(invoice, invoice_data)

        if invoice._need_ubl_cii_xml(invoice_data['invoice_edi_format']):
            builder = invoice.partner_id.commercial_partner_id._get_edi_builder(invoice_data['invoice_edi_format'])
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
                    'ubl_cii_format': invoice_data['invoice_edi_format'],
                    'builder': builder,
                }

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
        pdf_values = (not self.env.context.get('custom_template_facturx') and invoice.invoice_pdf_report_id) or \
            invoice_data.get('pdf_attachment_values') or invoice_data['proforma_pdf_attachment_values']
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
            except Exception:
                _logger.exception("Error while converting to PDF/A")

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
        pdf_values = invoice.invoice_pdf_report_id or invoice_data.get('pdf_attachment_values') or invoice_data['proforma_pdf_attachment_values']
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

    def _link_invoice_documents(self, invoices_data):
        # EXTENDS 'account'
        super()._link_invoice_documents(invoices_data)

        attachments_vals = [
            invoice_data.get('ubl_cii_xml_attachment_values')
            for invoice_data in invoices_data.values()
            if invoice_data.get('ubl_cii_xml_attachment_values')
        ]
        if attachments_vals:
            attachments = self.env['ir.attachment'].with_user(SUPERUSER_ID).create(attachments_vals)
            res_ids = attachments.mapped('res_id')
            self.env['account.move'].browse(res_ids).invalidate_recordset(fnames=['ubl_cii_xml_id', 'ubl_cii_xml_file'])

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
            and self._get_report(report_ref).report_name in custom_templates
        ):
            # Generate and embed Factur-X
            invoice = self.env['account.move'].browse(res_ids)
            if invoice.is_sale_document() and invoice.state == 'posted':
                pdf_stream = collected_streams[invoice.id]['stream']
                invoice_data = {'pdf_attachment_values': {'raw': pdf_stream.getvalue()}}
                self.env['account.move.send'].with_context(custom_template_facturx=True)._hook_invoice_document_after_pdf_report_render(invoice, invoice_data)
                collected_streams[invoice.id]['stream'] = io.BytesIO(invoice_data['pdf_attachment_values']['raw'])
        return collected_streams

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import re
from stdnum.fr import siret

from odoo import models, fields, api, _
from odoo.exceptions import ValidationError
from odoo.addons.account_edi_ubl_cii.models.account_edi_common import EAS_MAPPING
from odoo.addons.account.models.company import PEPPOL_DEFAULT_COUNTRIES


class ResPartner(models.Model):
    _inherit = 'res.partner'

    invoice_edi_format = fields.Selection(
        selection_add=[
            ('facturx', "Factur-X (CII)"),
            ('ubl_bis3', "BIS Billing 3.0"),
            ('xrechnung', "XRechnung CIUS"),
            ('nlcius', "NLCIUS"),
            ('ubl_a_nz', "BIS Billing 3.0 A-NZ"),
            ('ubl_sg', "BIS Billing 3.0 SG"),
        ],
    )
    is_ubl_format = fields.Boolean(compute='_compute_is_ubl_format')
    is_peppol_edi_format = fields.Boolean(compute='_compute_is_peppol_edi_format')  # TODO remove in master
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
            ('9923', "Albania VAT"),
            ('9922', "Andorra VAT"),
            ('0151', "Australia ABN"),
            ('9914', "Austria UID"),
            ('9915', "Austria VOKZ"),
            ('0208', "Belgian Company Registry"),
            ('9925', "Belgian VAT"),
            ('9924', "Bosnia and Herzegovina VAT"),
            ('9926', "Bulgaria VAT"),
            ('9934', "Croatia VAT"),
            ('9928', "Cyprus VAT"),
            ('9929', "Czech Republic VAT"),
            ('0096', "Denmark P"),
            ('0184', "Denmark CVR"),
            ('0198', "Denmark SE"),
            ('0191', "Estonia Company code"),
            ('9931', "Estonia VAT"),
            ('0037', "Finland LY-tunnus"),
            ('0216', "Finland OVT code"),
            ('0213', "Finland VAT"),
            ('0002', "France SIRENE"),
            ('0009', "France SIRET"),
            ('9957', "France VAT"),
            ('0204', "Germany Leitweg-ID"),
            ('9930', "Germany VAT"),
            ('9933', "Greece VAT"),
            ('9910', "Hungary VAT"),
            ('0196', "Iceland Kennitala"),
            ('9935', "Ireland VAT"),
            ('0211', "Italia Partita IVA"),
            ('0097', "Italia FTI"),
            ('0188', "Japan SST"),
            ('0221', "Japan IIN"),
            ('9939', "Latvia VAT"),
            ('9936', "Liechtenstein VAT"),
            ('0200', "Lithuania JAK"),
            ('9937', "Lithuania VAT"),
            ('9938', "Luxembourg VAT"),
            ('9942', "Macedonia VAT"),
            ('0230', "Malaysia"),
            ('9943', "Malta VAT"),
            ('9940', "Monaco VAT"),
            ('9941', "Montenegro VAT"),
            ('0106', "Netherlands KvK"),
            ('0190', "Netherlands OIN"),
            ('9944', "Netherlands VAT"),
            ('0192', "Norway Org.nr."),
            ('9945', "Poland VAT"),
            ('9946', "Portugal VAT"),
            ('9947', "Romania VAT"),
            ('9948', "Serbia VAT"),
            ('0195', "Singapore UEN"),
            ('9949', "Slovenia VAT"),
            ('9950', "Slovakia VAT"),
            ('9920', "Spain VAT"),
            ('0007', "Sweden Org.nr."),
            ('9955', "Sweden VAT"),
            ('9927', "Swiss VAT"),
            ('0183', "Swiss UIDB"),
            ('9952', "Turkey VAT"),
            ('9932', "United Kingdom VAT"),
            ('9959', "USA EIN"),
            ('0060', "DUNS Number"),
            ('0088', "EAN Location Code"),
            ('0130', "Directorates of the European Commission"),
            ('0135', "SIA Object Identifiers"),
            ('0142', "SECETI Object Identifiers"),
            ('0193', "UBL.BE party identifier"),
            ('0199', "Legal Entity Identifier (LEI)"),
            ('0201', "Codice Univoco Unità Organizzativa iPA"),
            ('0202', "Indirizzo di Posta Elettronica Certificata"),
            ('0209', "GS1 identification keys"),
            ('0210', "Codice Fiscale"),
            ('9913', "Business Registers Network"),
            ('9918', "S.W.I.F.T"),
            ('9919', "Kennziffer des Unternehmensregisters"),
            ('9951', "San Marino VAT"),
            ('9953', "Vatican VAT"),
        ]
    )

    @api.constrains('peppol_endpoint')
    def _check_peppol_fields(self):
        for partner in self:
            if partner.peppol_endpoint and partner.peppol_eas:
                error = self._build_error_peppol_endpoint(partner.peppol_eas, partner.peppol_endpoint)
                if error:
                    raise ValidationError(error)

    @api.model
    def _get_ubl_cii_formats(self):
        return list(self._get_ubl_cii_formats_info().keys())

    @api.model
    def _get_ubl_cii_formats_info(self):
        return {
            'ubl_bis3': {'countries': list(PEPPOL_DEFAULT_COUNTRIES), 'on_peppol': True, 'sequence': 200},
            'xrechnung': {'countries': ['DE'], 'on_peppol': True},
            'ubl_a_nz': {'countries': ['NZ', 'AU'], 'on_peppol': False},  # Not yet available through Odoo's Access Point, although it's a Peppol valid format
            'nlcius': {'countries': ['NL'], 'on_peppol': True},
            'ubl_sg': {'countries': ['SG'], 'on_peppol': False},  # Same.
            'facturx': {'countries': ['FR'], 'on_peppol': False},
        }

    @api.model
    def _get_ubl_cii_formats_by_country(self):
        formats_info = self._get_ubl_cii_formats_info()
        countries = {country for format_val in formats_info.values() for country in (format_val.get('countries') or [])}
        return {
            country_code: [
                format_key
                for format_key, format_val in formats_info.items() if country_code in (format_val.get('countries') or [])
            ]
            for country_code in countries
        }

    def _get_suggested_ubl_cii_edi_format(self):
        self.ensure_one()
        format_mapping = self._get_ubl_cii_formats_by_country()
        country_code = self.commercial_partner_id._deduce_country_code()
        if country_code in format_mapping:
            formats_by_country = format_mapping[country_code]
            # return the format with the smallest sequence
            if len(formats_by_country) == 1:
                return formats_by_country[0]
            else:
                formats_info = self._get_ubl_cii_formats_info()
                return min(formats_by_country, key=lambda e: formats_info[e].get('sequence', 100))  # we use a sequence of 100 by default
        return False

    def _get_suggested_peppol_edi_format(self):
        self.ensure_one()
        suggested_format = self.commercial_partner_id._get_suggested_ubl_cii_edi_format()
        return suggested_format if suggested_format in self.env['res.partner']._get_peppol_formats() else 'ubl_bis3'

    def _get_peppol_edi_format(self):
        self.ensure_one()
        return self.invoice_edi_format or self._get_suggested_peppol_edi_format()

    @api.model
    def _get_peppol_formats(self):
        formats_info = self._get_ubl_cii_formats_info()
        return [format_key for format_key, format_vals in formats_info.items() if format_vals.get('on_peppol')]

    def _peppol_eas_endpoint_depends(self):
        # field dependencies of methods _compute_peppol_endpoint() and _compute_peppol_eas()
        # because we need to extend depends in l10n modules
        return ['country_code', 'vat', 'company_registry']

    @api.depends_context('company')
    @api.depends('invoice_edi_format')
    def _compute_is_ubl_format(self):
        for partner in self:
            partner.is_ubl_format = partner.invoice_edi_format in self._get_ubl_cii_formats()

    @api.depends_context('company')
    @api.depends('invoice_edi_format')
    def _compute_is_peppol_edi_format(self):
        for partner in self:
            partner.is_peppol_edi_format = partner.invoice_edi_format in self._get_peppol_formats()

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

    @api.model
    def _get_edi_builder(self, invoice_edi_format):
        if invoice_edi_format == 'xrechnung':
            return self.env['account.edi.xml.ubl_de']
        if invoice_edi_format == 'facturx':
            return self.env['account.edi.xml.cii']
        if invoice_edi_format == 'ubl_a_nz':
            return self.env['account.edi.xml.ubl_a_nz']
        if invoice_edi_format == 'nlcius':
            return self.env['account.edi.xml.ubl_nl']
        if invoice_edi_format == 'ubl_bis3':
            return self.env['account.edi.xml.ubl_bis3']
        if invoice_edi_format == 'ubl_sg':
            return self.env['account.edi.xml.ubl_sg']

```

## File: models\__init__.py

```python
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
from . import account_move_send
from . import ir_actions_report
from . import res_partner

```

## File: views\res_partner_views.xml

```xml
<odoo>
    <record id="view_partner_property_form" model="ir.ui.view">
        <field name="name">res.partner.property.form.inherit</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@id='invoice_send_settings']" position="inside">
                <label for="peppol_eas" string="Peppol Address" invisible="1"/> <!-- TODO remove in master -->
                <div id="peppol_address" class="row" invisible="1"/> <!-- TODO remove in master -->
                <field name="peppol_eas"
                       placeholder="Peppol ID"
                       nolabel="1"
                       class="o_field_peppol_eas_selection"/>
                <field name="peppol_endpoint" nolabel="1" placeholder="Your endpoint"/>
            </xpath>
        </field>
    </record>
</odoo>

```

