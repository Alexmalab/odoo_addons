# Odoo Module: sale_edi_ubl

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': "Import electronic orders with UBL",
    'category': 'Sales/Sales',
    'description': """
Electronic ordering module
===========================

Allows to import formats: UBL Bis 3.
When uploading or pasting Files in order list view with order related data inside XML file or PDF
File with embedded xml data will allow seller to retrieve Order data from Files.
    """,
    'depends': ['sale', 'account_edi_ubl_cii'],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\sale_edi_common.py

```python
from markupsafe import Markup

from odoo import _, models


class SaleEdiCommon(models.AbstractModel):
    _name = 'sale.edi.common'
    _inherit = 'account.edi.common'
    _description = "Common functions for EDI orders"

    # -------------------------------------------------------------------------
    # Import order
    # -------------------------------------------------------------------------

    def _import_order_ubl(self, order, file_data):
        """ Common importing method to extract order data from file_data.

        :param order: Order to fill details from file_data.
        :param file_data: File data to extract order related data from.
        :return: True if there no exception while extraction.
        :rtype: Boolean
        """
        tree = file_data['xml_tree']

        # Update the order.
        logs = self._import_fill_order(order, tree)
        if order:
            body = Markup("<strong>%s</strong>") % \
                _("Format used to import the invoice: %s",
                  self.env['ir.model']._get(self._name).name)
            if logs:
                order._create_activity_set_details()
                body += Markup("<ul>%s</ul>") % \
                    Markup().join(Markup("<li>%s</li>") % l for l in logs)
            order.message_post(body=body)

        lines_with_products = order.order_line.filtered('product_id')
        # Recompute product price and discount according to sale price
        lines_with_products._compute_price_unit()
        lines_with_products._compute_discount()

        return True

    def _import_order_lines(self, order, tree, xpath):
        """ Import order lines from xml tree.

        :param order: Order to set order line on.
        :param tree: Xml tree to extract OrderLine from.
        :param xpath: Xpath for order line items.
        :return: Logging information related orderlines details.
        :rtype: List
        """
        logs = []
        lines_values = []
        for line_tree in tree.iterfind(xpath):
            line_values = self._retrieve_line_vals(line_tree)
            line_values = {
                **line_values,
                'product_uom_qty': line_values['quantity'],
                'product_uom': line_values['product_uom_id'],
            }
            del line_values['quantity']
            # To do: rename product_uom field to `product_uom_id` of sale.order.line
            del line_values['product_uom_id']
            if not line_values['product_id']:
                logs += [_("Could not retrieve product for line '%s'", line_values['name'])]
            # To do: rename tax_id field to `tax_ids` of sale.order.line
            line_values['tax_id'], tax_logs = self._retrieve_taxes(
                order, line_values, 'sale',
            )
            logs += tax_logs
            lines_values += self._retrieve_line_charges(order, line_values, line_values['tax_id'])
            if not line_values['product_uom']:
                line_values.pop('product_uom')  # if no uom, pop it so it's inferred from the product_id
            lines_values.append(line_values)

        return lines_values, logs

    def _import_payment_term_id(self, order, tree, xapth):
        """ Return payment term from given tree. """
        payment_term_note = self._find_value(xapth, tree)
        if not payment_term_note:
            return False

        return self.env['account.payment.term'].search([
            *self.env['account.payment.term']._check_company_domain(order.company_id),
            ('name', '=', payment_term_note)
        ], limit=1)

    def _import_delivery_partner(self, order, name, phone, email):
        """ Import delivery address from details if not found then log details."""
        logs = []
        dest_partner = self.env['res.partner'].with_company(
            order.company_id
        )._retrieve_partner(name=name, phone=phone, email=email)
        if not dest_partner:
            partner_detaits_str = self._get_partner_detail_str(name, phone, email)
            logs.append(_("Could not retrieve Delivery Address with Details: { %s }", partner_detaits_str))

        return dest_partner, logs

    def _import_partner(self, company_id, name, phone, email, vat, **kwargs):
        """ Override of edi.mixin to set current user partner if there is no matching partner
        found and log details related to partner."""
        partner, logs = super()._import_partner(company_id, name, phone, email, vat, **kwargs)
        if not partner:
            partner_detaits_str = self._get_partner_detail_str(name, phone, email, vat)
            if not vat:
                logs.append(_("Insufficient details to extract Customer: { %s }", partner_detaits_str))
            else:
                logs.append(_("Could not retrive Customer with Details: { %s }", partner_detaits_str))

        return partner, logs

    def _get_partner_detail_str(self, name, phone=False, email=False, vat=False):
        """ Return partner details string to help user find or create proper contact with details.
        """
        partner_details = _("Name: %(name)s, Vat: %(vat)s", name=name, vat=vat)
        if phone:
            partner_details += _(", Phone: %(phone)s", phone=phone)
        if email:
            partner_details += _(", Email: %(email)s", email=email)

        return partner_details

```

## File: models\sale_edi_xml_ubl_bis3.py

```python
from odoo import models, Command


class SaleEdiXmlUBLBIS3(models.AbstractModel):
    _name = 'sale.edi.xml.ubl_bis3'
    _inherit = ['sale.edi.common', 'account.edi.xml.ubl_bis3']
    _description = "UBL BIS Ordering 3.0"

    # -------------------------------------------------------------------------
    # Import order
    # -------------------------------------------------------------------------

    def _import_fill_order(self, order, tree):
        """ Fill order details by extracting details from xml tree.

        param order: Order to fill details from xml tree.
        param tree: Xml tree to extract details.
        :return: list of logs to add warnig and information about data from xml.
        """
        logs = []
        order_values = {}
        partner, partner_logs = self._import_partner(
            order.company_id,
            **self._import_retrieve_partner_vals(tree, "BuyerCustomer"),
        )
        if partner:
            order_values['partner_id'] = partner.id
        delivery_partner, delivery_partner_logs = self._import_delivery_partner(
            order,
            **self._import_retrieve_delivery_vals(tree),
        )
        if delivery_partner:
            order_values['partner_shipping_id'] = delivery_partner.id
        order_values['currency_id'], currency_logs = self._import_currency(tree, './/{*}DocumentCurrencyCode')

        order_values['date_order'] = tree.findtext('./{*}IssueDate')
        order_values['client_order_ref'] = tree.findtext('./{*}ID')
        order_values['note'] = self._import_description(tree, xpaths=['./{*}Note'])
        order_values['origin'] = tree.findtext('./{*}OriginatorDocumentReference/{*}ID')
        order_values['payment_term_id'] = self._import_payment_term_id(order, tree, './/cac:PaymentTerms/cbc:Note')

        allowance_charges_line_vals, allowance_charges_logs = self._import_document_allowance_charges(tree, order, 'sale')
        lines_vals, line_logs = self._import_order_lines(order, tree, './{*}OrderLine/{*}LineItem')
        lines_vals += allowance_charges_line_vals

        order_values = {
            **order_values,
            'order_line': [Command.create(line_vals) for line_vals in lines_vals],
        }
        order.write(order_values)
        logs += partner_logs + delivery_partner_logs + currency_logs + line_logs + allowance_charges_logs

        return logs

    def _import_retrieve_delivery_vals(self, tree):
        """ Returns a dict of values that will be used to retrieve the delivery address. """
        return {
            'phone': self._find_value('.//cac:Delivery/cac:DeliveryParty//cbc:Telephone', tree),
            'email': self._find_value('.//cac:Delivery/cac:DeliveryParty//cbc:ElectronicMail', tree),
            'name': self._find_value('.//cac:Delivery/cac:DeliveryParty//cbc:Name', tree),
        }

    def _get_line_xpaths(self, document_type=None, qty_factor=1):
        # Override account.edi.xml.ubl_bis3
        return {
            **super()._get_line_xpaths(),
            'delivered_qty': ('./{*}Quantity'),
        }

```

## File: models\sale_order.py

```python
from odoo import _, api, models, Command


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    def _get_order_edi_decoder(self, file_data):
        """ Override of sale to add edi decoder for xml files.

        :param dict file_data: File data to decode.
        :return function: Function with decoding capibility `_import_order_ubl` for different xml
        formats.
        """
        if file_data['type'] == 'xml':
            ubl_cii_xml_builder = self._get_order_ubl_builder_from_xml_tree(file_data['xml_tree'])
            if ubl_cii_xml_builder is not None:
                return ubl_cii_xml_builder._import_order_ubl

        return super()._get_order_edi_decoder(file_data)

    @api.model
    def _get_order_ubl_builder_from_xml_tree(self, tree):
        """ Return sale order ubl builder with decording capibily to given tree

        :param xml tree: xml tree to find builder.
        :return class: class object of builder for given tree if found else none.
        """
        customization_id = tree.find('{*}CustomizationID')
        if customization_id is not None:
            if customization_id.text == 'urn:fdc:peppol.eu:poacc:trns:order:3':
                return self.env['sale.edi.xml.ubl_bis3']

    def _create_activity_set_details(self):
        """ Create activity on sale order to set details.

        :return: None.
        """
        activity_message = _("Some information could not be imported")
        self.activity_schedule(
            'mail.mail_activity_data_todo',
            user_id=self.env.user.id,
            note=activity_message,
        )

    @api.model
    def _get_line_vals_list(self, lines_vals):
        """ Get sale order line values list.

        :param list line_vals: List of values [name, qty, price, tax].
        :return: List of dict values.
        """

        return [{
            'sequence': 0,  # be sure to put these lines above the 'real' order lines
            'name': name,
            'product_uom_qty': quantity,
            'price_unit': price_unit,
            'tax_id': [Command.set(tax_ids)],
        } for name, quantity, price_unit, tax_ids in lines_vals]

```

## File: models\__init__.py

```python
from . import sale_edi_common
from . import sale_edi_xml_ubl_bis3
from . import sale_order

```

