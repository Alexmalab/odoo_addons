# Odoo Module: l10n_ro_edi_stock_batch

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Romania - E-Transport Batch Pickings',
    'version': '1.0',
    'category': 'Accounting/Localizations/EDI',
    'description': """
E-Transport implementation for Batch Pickings in Romania
    """,
    'depends': ['l10n_ro_edi_stock', 'stock_picking_batch'],
    'auto_install': True,
    'data': [
        'views/stock_picking_batch_views.xml',

        'report/report_picking_batch.xml',
    ],
    'installable': True,
    'license': "LGPL-3",
}

```

## File: models\l10n_ro_edi_stock_document.py

```python
from odoo import models, fields


class L10nRoEdiStockDocument(models.Model):
    _inherit = 'l10n_ro_edi.document'

    batch_id = fields.Many2one(comodel_name='stock.picking.batch')

```

## File: models\stock_picking.py

```python
from odoo import api, models


class Picking(models.Model):
    _inherit = 'stock.picking'

    @api.depends('batch_id', 'company_id')
    def _compute_l10n_ro_edi_stock_enable(self):
        # OVERRIDES 'l10n_ro_edi_stock'
        for picking in self:
            picking.l10n_ro_edi_stock_enable = not picking.batch_id and picking.company_id.country_id.code == 'RO'

    @api.model
    def _l10n_ro_edi_stock_validate_carrier_filter(self, picking):
        # OVERRIDE l10n_ro_edi_stock

        # Override for when the batch picking calls this function to validate the carriers
        validate_carrier = self.env.context.get('l10n_ro_edi_stock_validate_carrier', False)

        return picking.company_id.account_fiscal_country_id.code == 'RO' and (picking.l10n_ro_edi_stock_enable or validate_carrier)

```

## File: models\stock_picking_batch.py

```python
import markupsafe
import requests

from odoo import fields, models, api, _
from odoo.exceptions import UserError
from odoo.addons.l10n_ro_edi_stock.models.stock_picking import OPERATION_TYPES, OPERATION_SCOPES, OPERATION_TYPE_TO_ALLOWED_SCOPE_CODES, LOCATION_TYPES, LOCATION_TYPE_MAP, BORDER_CROSSING_POINTS, CUSTOMS_OFFICES, STATE_CODES
from odoo.addons.l10n_ro_edi_stock.models.l10n_ro_edi_stock_document import DOCUMENT_STATES
from odoo.addons.l10n_ro_edi_stock.models.etransport_api import ETransportAPI


class StockPickingBatch(models.Model):
    _inherit = 'stock.picking.batch'

    # Document fields
    l10n_ro_edi_stock_document_ids = fields.One2many(comodel_name='l10n_ro_edi.document', inverse_name='batch_id')
    l10n_ro_edi_stock_document_uit = fields.Char(compute='_compute_l10n_ro_edi_stock_current_document_uit', string="eTransport UIT")
    l10n_ro_edi_stock_state = fields.Selection(
        selection=DOCUMENT_STATES,
        compute='_compute_l10n_ro_edi_stock_current_document_state',
        string="eTransport Status",
        store=True,
    )

    # Data fields
    l10n_ro_edi_stock_operation_type = fields.Selection(selection=OPERATION_TYPES, string="eTransport Operation Type")
    l10n_ro_edi_stock_available_operation_scopes = fields.Char(compute='_compute_l10n_ro_edi_stock_available_operation_scopes')
    l10n_ro_edi_stock_operation_scope = fields.Selection(selection=OPERATION_SCOPES, string="Operation Scope")

    l10n_ro_edi_stock_vehicle_number = fields.Char(string="Vehicle Number", size=20)
    l10n_ro_edi_stock_trailer_1_number = fields.Char(string="Trailer 1 Number", size=20)
    l10n_ro_edi_stock_trailer_2_number = fields.Char(string="Trailer 2 Number", size=20)

    l10n_ro_edi_stock_available_start_loc_types = fields.Char(compute='_compute_l10n_ro_edi_stock_available_location_types')
    l10n_ro_edi_stock_start_loc_type = fields.Selection(
        selection=LOCATION_TYPES,
        string="Start Location Type",
        compute='_compute_l10n_ro_edi_stock_default_location_type',
        store=True,
        readonly=False,
    )

    l10n_ro_edi_stock_available_end_loc_types = fields.Char(compute='_compute_l10n_ro_edi_stock_available_location_types')
    l10n_ro_edi_stock_end_loc_type = fields.Selection(
        selection=LOCATION_TYPES,
        string="End Location Type",
        compute='_compute_l10n_ro_edi_stock_default_location_type',
        store=True,
        readonly=False,
    )

    # Data fields for every location type
    l10n_ro_edi_stock_start_bcp = fields.Selection(selection=BORDER_CROSSING_POINTS, string="Start Border Crossing Point")
    l10n_ro_edi_stock_start_customs_office = fields.Selection(selection=CUSTOMS_OFFICES, string="Start Customs Office")

    l10n_ro_edi_stock_end_bcp = fields.Selection(selection=BORDER_CROSSING_POINTS, string="End Border Crossing Point")
    l10n_ro_edi_stock_end_customs_office = fields.Selection(selection=CUSTOMS_OFFICES, string="End Customs Office")

    l10n_ro_edi_stock_remarks = fields.Text(string="Remarks")

    # View control fields
    l10n_ro_edi_stock_enable = fields.Boolean(compute='_compute_l10n_ro_edi_stock_enable')
    l10n_ro_edi_stock_enable_send = fields.Boolean(compute='_compute_l10n_ro_edi_stock_enable_send')
    l10n_ro_edi_stock_enable_fetch = fields.Boolean(compute='_compute_l10n_ro_edi_stock_enable_fetch')
    l10n_ro_edi_stock_enable_amend = fields.Boolean(compute='_compute_l10n_ro_edi_stock_enable_amend')

    l10n_ro_edi_stock_fields_readonly = fields.Boolean(compute='_compute_l10n_ro_edi_stock_fields_readonly')

    ################################################################################
    # Onchange Methods
    ################################################################################

    @api.onchange('l10n_ro_edi_stock_operation_type')
    def _l10n_ro_edi_stock_reset_variable_selection_fields(self):
        self.l10n_ro_edi_stock_operation_scope = False

        # the 'location' value is always valid, regardless of which operation type is chosen
        self.l10n_ro_edi_stock_start_loc_type = 'location'
        self.l10n_ro_edi_stock_end_loc_type = 'location'

    ################################################################################
    # Compute Methods
    ################################################################################

    @api.depends('company_id.account_fiscal_country_id.code')
    def _compute_l10n_ro_edi_stock_default_location_type(self):
        for batch in self:
            if batch.company_id.account_fiscal_country_id.code == 'RO':
                if not batch.l10n_ro_edi_stock_start_loc_type:
                    batch.l10n_ro_edi_stock_start_loc_type = 'location'
                else:
                    batch.l10n_ro_edi_stock_start_loc_type = batch.l10n_ro_edi_stock_start_loc_type

                if not batch.l10n_ro_edi_stock_end_loc_type:
                    batch.l10n_ro_edi_stock_end_loc_type = 'location'
                else:
                    batch.l10n_ro_edi_stock_start_loc_type = batch.l10n_ro_edi_stock_start_loc_type
            else:
                batch.l10n_ro_edi_stock_start_loc_type = False
                batch.l10n_ro_edi_stock_end_loc_type = False

    @api.depends('l10n_ro_edi_stock_operation_type')
    def _compute_l10n_ro_edi_stock_available_operation_scopes(self):
        for batch in self:
            if batch.l10n_ro_edi_stock_operation_type:
                allowed_scopes = OPERATION_TYPE_TO_ALLOWED_SCOPE_CODES.get(batch.l10n_ro_edi_stock_operation_type, ("9999",))
            else:
                allowed_scopes = [c for c, _dummy in OPERATION_SCOPES]

            batch.l10n_ro_edi_stock_available_operation_scopes = ','.join(allowed_scopes)

    @api.depends('l10n_ro_edi_stock_operation_type')
    def _compute_l10n_ro_edi_stock_available_location_types(self):
        for batch in self:
            batch.l10n_ro_edi_stock_available_start_loc_types = self.env['stock.picking']._l10n_ro_edi_stock_get_available_location_types(batch.l10n_ro_edi_stock_operation_type, 'start')
            batch.l10n_ro_edi_stock_available_end_loc_types = self.env['stock.picking']._l10n_ro_edi_stock_get_available_location_types(batch.l10n_ro_edi_stock_operation_type, 'end')

    @api.depends('l10n_ro_edi_stock_document_ids', 'company_id.account_fiscal_country_id.code')
    def _compute_l10n_ro_edi_stock_current_document_state(self):
        for batch in self:
            if batch.company_id.account_fiscal_country_id.code == 'RO' and (document := batch._l10n_ro_edi_stock_get_current_document()):
                batch.l10n_ro_edi_stock_state = document.state
            else:
                batch.l10n_ro_edi_stock_state = False

    @api.depends('l10n_ro_edi_stock_document_ids', 'company_id.account_fiscal_country_id.code')
    def _compute_l10n_ro_edi_stock_current_document_uit(self):
        for batch in self:
            if batch.company_id.account_fiscal_country_id.code == 'RO' and (document := batch._l10n_ro_edi_stock_get_current_document()):
                batch.l10n_ro_edi_stock_document_uit = document.l10n_ro_edi_stock_uit
            else:
                batch.l10n_ro_edi_stock_document_uit = False

    @api.depends('company_id.account_fiscal_country_id.code')
    def _compute_l10n_ro_edi_stock_enable(self):
        for batch in self:
            batch.l10n_ro_edi_stock_enable = batch.company_id.account_fiscal_country_id.code == 'RO'

    @api.depends('l10n_ro_edi_stock_enable', 'state', 'l10n_ro_edi_stock_state')
    def _compute_l10n_ro_edi_stock_enable_send(self):
        for batch in self:
            batch.l10n_ro_edi_stock_enable_send = (batch.l10n_ro_edi_stock_enable
                                                   and batch.state != 'draft'
                                                   and batch.l10n_ro_edi_stock_state in (False, 'stock_sending_failed')
                                                   and not batch._l10n_ro_edi_stock_get_last_document('stock_validated'))

    @api.depends('l10n_ro_edi_stock_enable', 'state', 'l10n_ro_edi_stock_state')
    def _compute_l10n_ro_edi_stock_enable_fetch(self):
        for batch in self:
            batch.l10n_ro_edi_stock_enable_fetch = batch.l10n_ro_edi_stock_enable and batch.l10n_ro_edi_stock_state == 'stock_sent'

    @api.depends('l10n_ro_edi_stock_state')
    def _compute_l10n_ro_edi_stock_enable_amend(self):
        for batch in self:
            batch.l10n_ro_edi_stock_enable_amend = (batch.l10n_ro_edi_stock_enable
                                                    and batch.l10n_ro_edi_stock_state == 'stock_validated'
                                                    or (batch.l10n_ro_edi_stock_state == 'stock_sending_failed'
                                                        and batch._l10n_ro_edi_stock_get_last_document('stock_validated')))

    @api.depends('l10n_ro_edi_stock_state')
    def _compute_l10n_ro_edi_stock_fields_readonly(self):
        for batch in self:
            batch.l10n_ro_edi_stock_fields_readonly = batch.l10n_ro_edi_stock_state == 'stock_sent'

    ################################################################################
    # Validation methods
    ################################################################################

    def action_done(self):
        # EXTENDS 'stock_picking_batch'
        self.ensure_one()
        self._check_company()

        self.picking_ids.with_context(l10n_ro_edi_stock_validate_carrier=True)._l10n_ro_edi_stock_validate_carrier()

        # Carrier should be the same on all pickings
        first_carrier = self.picking_ids[0].carrier_id
        if any(picking.carrier_id != first_carrier for picking in self.picking_ids):
            raise UserError(_("All Pickings in a Batch Transfer should have the same Carrier"))

        # Commercial partner should be the same on all pickings
        first_commercial_partner = self.picking_ids[0].partner_id.commercial_partner_id
        if any(picking.partner_id.commercial_partner_id != first_commercial_partner for picking in self.picking_ids):
            raise UserError(_("All Pickings in a Batch Transfer should have the same Commercial Partner"))

        return super().action_done()

    def _l10n_ro_edi_stock_validate_fetch_data(self, errors=None):
        if errors is None:
            errors = []
        self.ensure_one()

        if not self.company_id.l10n_ro_edi_access_token:
            errors.append(_('Romanian access token not found. Please generate or fill it in the settings.'))
            return errors

        match self.l10n_ro_edi_stock_state:
            case 'stock_sending_failed':
                if not self._l10n_ro_edi_stock_get_last_document('stock_validated'):
                    errors.append(_("This document has not been successfully sent yet because it contains errors."))
                else:
                    errors.append(_("This document has not been corrected yet because it contains errors."))
            case 'stock_validated':
                errors.append(_("This document has already been successfully sent to anaf."))

        return errors

    ################################################################################
    # Actions
    ################################################################################

    def action_l10n_ro_edi_stock_send_etransport(self):
        self.ensure_one()

        send_type = self.env.context.get('l10n_ro_edi_stock_send_type', 'send')
        self._l10n_ro_edi_stock_send_etransport_document(send_type=send_type)

    def action_l10n_ro_edi_stock_fetch_status(self):
        self._l10n_ro_edi_stock_fetch_document_status()

    ################################################################################
    # Document Helpers
    ################################################################################

    def _l10n_ro_edi_stock_get_current_document(self):
        self.ensure_one()
        return self.l10n_ro_edi_stock_document_ids.sorted()[0] if self.l10n_ro_edi_stock_document_ids else None

    def _l10n_ro_edi_stock_get_all_documents(self, states):
        self.ensure_one()

        if isinstance(states, str):
            states = [states]

        return self.l10n_ro_edi_stock_document_ids.filtered(lambda doc: doc.state in states)

    def _l10n_ro_edi_stock_get_last_document(self, state):
        self.ensure_one()
        documents_in_state = self.l10n_ro_edi_stock_document_ids.filtered(lambda doc: doc.state == state).sorted()

        return documents_in_state and documents_in_state[0]

    def _l10n_ro_edi_stock_create_document_stock_sent(self, values: dict[str, object]):
        self.ensure_one()
        document = self.env['l10n_ro_edi.document'].create({
            'batch_id': self.id,
            'state': 'stock_sent',
            'l10n_ro_edi_stock_load_id': values['l10n_ro_edi_stock_load_id'],
            'l10n_ro_edi_stock_uit': values['l10n_ro_edi_stock_uit'],
        })

        document.attachment_id = self.env['stock.picking']._l10n_ro_edi_stock_create_attachment({
            'name': self.name,
            'res_id': document.id,
            'raw': values['raw_xml'],
        })

        return document

    def _l10n_ro_edi_stock_create_document_stock_sending_failed(self, values: dict[str, object]):
        self.ensure_one()
        document = self.env['l10n_ro_edi.document'].create({
            'batch_id': self.id,
            'state': 'stock_sending_failed',
            'message': values['message'],
            'l10n_ro_edi_stock_load_id': values.get('l10n_ro_edi_stock_load_id'),
            'l10n_ro_edi_stock_uit': values.get('l10n_ro_edi_stock_uit'),
        })

        if 'raw_xml' in values:
            # when an error is thrown during data validation there will be no 'raw_xml'
            document.attachment_id = self.env['stock.picking']._l10n_ro_edi_stock_create_attachment({
                'name': self.name,
                'res_id': document.id,
                'raw': values['raw_xml'],
            })

        return document

    def _l10n_ro_edi_stock_create_document_stock_validated(self, values: dict[str, object]):
        self.ensure_one()
        document = self.env['l10n_ro_edi.document'].create({
            'batch_id': self.id,
            'state': 'stock_validated',
            'l10n_ro_edi_stock_load_id': values['l10n_ro_edi_stock_load_id'],
            'l10n_ro_edi_stock_uit': values['l10n_ro_edi_stock_uit'],
        })

        document.attachment_id = self.env['stock.picking']._l10n_ro_edi_stock_create_attachment({
            'name': self.name,
            'res_id': document.id,
            'raw': values['raw_xml'],
        })

        return document

    ################################################################################
    # Send Logic
    ################################################################################

    def _l10n_ro_edi_stock_send_etransport_document(self, send_type: str):
        """
        Send the eTransport document to anaf
        :param send_type: 'send' (initial sending of document) | 'amend' (correct the already sent document)
        """
        self.ensure_one()

        data = {
            'partner_id': self.picking_ids[0].partner_id,
            'transport_partner_id': self.picking_ids[0].carrier_id.l10n_ro_edi_stock_partner_id,
            'company_id': self.company_id,
            'scheduled_date': self.scheduled_date,
            'name': self.name,
            'send_type': send_type,
            'l10n_ro_edi_stock_operation_type': self.l10n_ro_edi_stock_operation_type,
            'l10n_ro_edi_stock_operation_scope': self.l10n_ro_edi_stock_operation_scope,
            'stock_move_ids': self.move_ids,
            'l10n_ro_edi_stock_vehicle_number': self.l10n_ro_edi_stock_vehicle_number,
            'l10n_ro_edi_stock_trailer_1_number': self.l10n_ro_edi_stock_trailer_1_number,
            'l10n_ro_edi_stock_trailer_2_number': self.l10n_ro_edi_stock_trailer_2_number,
            'l10n_ro_edi_stock_start_loc_type': self.l10n_ro_edi_stock_start_loc_type,
            'l10n_ro_edi_stock_end_loc_type': self.l10n_ro_edi_stock_end_loc_type,
            'l10n_ro_edi_stock_remarks': self.l10n_ro_edi_stock_remarks,
            'picking_type_id': self.picking_type_id,
            'l10n_ro_edi_stock_start_bcp': self.l10n_ro_edi_stock_start_bcp,
            'l10n_ro_edi_stock_end_bcp': self.l10n_ro_edi_stock_end_bcp,
            'l10n_ro_edi_stock_start_customs_office': self.l10n_ro_edi_stock_start_customs_office,
            'l10n_ro_edi_stock_end_customs_office': self.l10n_ro_edi_stock_end_customs_office,
            'l10n_ro_edi_stock_document_uit': self.l10n_ro_edi_stock_document_uit,
        }

        if errors := self.env['stock.picking']._l10n_ro_edi_stock_validate_data(data=data):
            self._l10n_ro_edi_stock_get_all_documents('stock_sending_failed').unlink()
            document_values = {'message': '\n'.join(errors)}

            if send_type == 'amend':
                last_sent_document = self._l10n_ro_edi_stock_get_last_document('stock_validated')
                document_values |= {
                    'l10n_ro_edi_stock_load_id': last_sent_document.l10n_ro_edi_stock_load_id,
                    'l10n_ro_edi_stock_uit': last_sent_document.l10n_ro_edi_stock_uit,
                    'raw_xml': last_sent_document.attachment_id.raw,
                }

            self._l10n_ro_edi_stock_create_document_stock_sending_failed(document_values)
            return

        raw_xml = markupsafe.Markup("<?xml version='1.0' encoding='UTF-8'?>\n") + self.env['ir.qweb']._render(
            'l10n_ro_edi_stock.l10n_ro_template_etransport',
            values=self.env['stock.picking']._l10n_ro_edi_stock_get_template_data(data=data),
        )

        result = ETransportAPI().upload_data(company_id=self.company_id, data=raw_xml)

        if 'error' in result:
            self._l10n_ro_edi_stock_get_all_documents('stock_sending_failed').unlink()
            document_values = {'message': result['error'], 'raw_xml': raw_xml}

            if send_type == 'amend':
                last_sent_document = self._l10n_ro_edi_stock_get_last_document('stock_validated')
                document_values |= {
                    'l10n_ro_edi_stock_load_id': last_sent_document.l10n_ro_edi_stock_load_id,
                    'l10n_ro_edi_stock_uit': last_sent_document.l10n_ro_edi_stock_uit,
                }

            self._l10n_ro_edi_stock_create_document_stock_sending_failed(document_values)
        else:
            self._l10n_ro_edi_stock_get_all_documents({'stock_sending_failed', 'stock_sent'}).unlink()

            content = result['content']

            if send_type == 'send':
                uit = content['UIT']
            else:
                last_validated = self._l10n_ro_edi_stock_get_last_document('stock_validated')
                uit = last_validated.l10n_ro_edi_stock_uit
                raw_xml = last_validated.attachment_id.raw

            self._l10n_ro_edi_stock_create_document_stock_sent({
                'l10n_ro_edi_stock_load_id': content['index_incarcare'],
                'l10n_ro_edi_stock_uit': uit,
                'raw_xml': raw_xml,
            })

    def _l10n_ro_edi_stock_fetch_document_status(self):
        session = requests.Session()
        documents_to_delete = self.env['l10n_ro_edi.document']
        to_fetch = self.filtered(lambda b: b.l10n_ro_edi_stock_state == 'stock_sent')

        for batch in to_fetch:
            current_sending_document = batch.l10n_ro_edi_stock_document_ids.filtered(lambda doc: doc.state == 'stock_sent')[0]

            if errors := batch._l10n_ro_edi_stock_validate_fetch_data():
                documents_to_delete |= batch._l10n_ro_edi_stock_get_all_documents('stock_sending_failed')
                batch._l10n_ro_edi_stock_create_document_stock_sending_failed({
                    'message': '\n'.join(errors),
                    'l10n_ro_edi_stock_load_id': current_sending_document.l10n_ro_edi_stock_load_id,
                    'l10n_ro_edi_stock_uit': current_sending_document.l10n_ro_edi_stock_uit,
                    'raw_xml': current_sending_document.attachment_id.raw,
                })
                continue

            result = ETransportAPI().get_status(
                company_id=batch.company_id,
                document_load_id=current_sending_document.l10n_ro_edi_stock_load_id,
                session=session,
            )

            if 'error' in result:
                documents_to_delete |= batch._l10n_ro_edi_stock_get_all_documents('stock_sending_failed')
                batch._l10n_ro_edi_stock_create_document_stock_sending_failed({
                    'message': result['error'],
                    'l10n_ro_edi_stock_load_id': current_sending_document.l10n_ro_edi_stock_load_id,
                    'l10n_ro_edi_stock_uit': current_sending_document.l10n_ro_edi_stock_uit,
                    'raw_xml': current_sending_document.attachment_id.raw,
                })
            else:
                documents_to_delete |= batch._l10n_ro_edi_stock_get_all_documents(('stock_sent', 'stock_sending_failed'))
                new_document_data = {
                    'l10n_ro_edi_stock_load_id': current_sending_document.l10n_ro_edi_stock_load_id,
                    'l10n_ro_edi_stock_uit': current_sending_document.l10n_ro_edi_stock_uit,
                    'raw_xml': current_sending_document.attachment_id.raw,
                }
                match state := result['content']['stare']:
                    case 'ok':
                        batch._l10n_ro_edi_stock_create_document_stock_validated(new_document_data)
                    case 'in prelucrare':
                        # Document is still being validated
                        batch._l10n_ro_edi_stock_create_document_stock_sent(new_document_data)
                    case 'XML cu erori nepreluat de sistem':
                        new_document_data['message'] = _("XML contains errors.")
                        batch._l10n_ro_edi_stock_create_document_stock_sending_failed(new_document_data)
                    case _:
                        batch._l10n_ro_edi_stock_report_unhandled_document_state(state)

        documents_to_delete.unlink()

    ################################################################################
    # Misc helpers
    ################################################################################

    def _l10n_ro_edi_stock_report_unhandled_document_state(self, state: str):
        self.ensure_one()
        self.message_post(body=_("Unhandled eTransport document state: %(state)s", state=state))

```

## File: models\__init__.py

```python
from . import l10n_ro_edi_stock_document
from . import stock_picking_batch
from . import stock_picking

```

## File: report\report_picking_batch.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo>
    <template id="l10n_ro_edi_stock_report_picking_batch" inherit_id="stock_picking_batch.report_picking_batch">
        <xpath expr="//div/h3[contains(text(), 'Summary:')]/../.." position="after">
            <div t-if="o.l10n_ro_edi_stock_enable and o.l10n_ro_edi_stock_document_uit" name="div_etransport_uit">
                <strong>eTransport UIT:</strong>
                <span t-field="o.l10n_ro_edi_stock_document_uit">3U3H3P4C2U5Y8N20</span>
            </div>
        </xpath>
    </template>
</odoo>

```

## File: views\stock_picking_batch_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo>
   <record id="l10n_ro_edi_stock_view_batch_form" model="ir.ui.view">
      <field name="name">stock.picking.batch.form.inherit.l10n_ro_edi_stock</field>
      <field name="model">stock.picking.batch</field>
      <field name="inherit_id" ref="stock_picking_batch.stock_picking_batch_form"/>
      <field name="arch" type="xml">
         <xpath expr="//button[@name='action_open_label_layout']" position="after">
             <field name="l10n_ro_edi_stock_enable_send" invisible="1"/>
             <field name="l10n_ro_edi_stock_enable_fetch" invisible="1"/>
             <field name="l10n_ro_edi_stock_enable_amend" invisible="1"/>

             <button name="action_l10n_ro_edi_stock_send_etransport"
                     string="Send eTransport"
                     type="object"
                     context="{'l10n_ro_edi_stock_send_type': 'send'}"
                     invisible="not l10n_ro_edi_stock_enable_send"/>
             <button name="action_l10n_ro_edi_stock_send_etransport"
                     string="Amend eTransport"
                     type="object"
                     context="{'l10n_ro_edi_stock_send_type': 'amend'}"
                     invisible="not l10n_ro_edi_stock_enable_amend"/>
             <button name="action_l10n_ro_edi_stock_fetch_status" string="Fetch Status" type="object" invisible="not l10n_ro_edi_stock_enable_fetch"/>
         </xpath>

         <xpath expr="//field[@name='scheduled_date']" position="after">
             <field name="l10n_ro_edi_stock_state" invisible="1"/>
             <field name="l10n_ro_edi_stock_state" invisible="state == 'draft' or not l10n_ro_edi_stock_state" readonly="1"/>
         </xpath>

         <xpath expr="//page[@name='page_transfers']" position="after">
             <field name="l10n_ro_edi_stock_enable" invisible="1"/>

             <page name="etransport" string="eTransport" invisible="not l10n_ro_edi_stock_enable">
                 <field name="l10n_ro_edi_stock_available_operation_scopes" invisible="1"/>
                 <field name="l10n_ro_edi_stock_state" invisible="1"/>
                 <field name="l10n_ro_edi_stock_fields_readonly" invisible="1"/>

                 <group>
                     <group string="General">
                         <field name="l10n_ro_edi_stock_operation_type" readonly="l10n_ro_edi_stock_fields_readonly"/>
                         <field name="l10n_ro_edi_stock_operation_scope"
                                widget="dynamic_selection"
                                options="{'available_field': 'l10n_ro_edi_stock_available_operation_scopes'}"
                                readonly="l10n_ro_edi_stock_fields_readonly"/>
                         <field name="l10n_ro_edi_stock_remarks" readonly="l10n_ro_edi_stock_fields_readonly"/>
                     </group>

                     <group string="Transport">
                         <field name="l10n_ro_edi_stock_vehicle_number" readonly="l10n_ro_edi_stock_fields_readonly"/>
                         <field name="l10n_ro_edi_stock_trailer_1_number" readonly="l10n_ro_edi_stock_fields_readonly"/>
                         <field name="l10n_ro_edi_stock_trailer_2_number" readonly="l10n_ro_edi_stock_fields_readonly"/>
                     </group>

                     <group string="Start Location">
                         <field name="l10n_ro_edi_stock_available_start_loc_types" invisible="1"/>
                         <field name="l10n_ro_edi_stock_start_loc_type"
                                widget="dynamic_selection"
                                options="{'available_field': 'l10n_ro_edi_stock_available_start_loc_types'}"
                                readonly="l10n_ro_edi_stock_fields_readonly"/>

                         <field name="l10n_ro_edi_stock_start_bcp" readonly="l10n_ro_edi_stock_fields_readonly" invisible="l10n_ro_edi_stock_start_loc_type != 'bcp'" />
                         <field name="l10n_ro_edi_stock_start_customs_office" readonly="l10n_ro_edi_stock_fields_readonly" invisible="l10n_ro_edi_stock_start_loc_type != 'customs'"/>
                     </group>

                     <group string="End Location">
                         <field name="l10n_ro_edi_stock_available_end_loc_types" invisible="1"/>
                         <field name="l10n_ro_edi_stock_end_loc_type"
                                widget="dynamic_selection"
                                options="{'available_field': 'l10n_ro_edi_stock_available_end_loc_types'}"
                                readonly="l10n_ro_edi_stock_fields_readonly"/>

                         <field name="l10n_ro_edi_stock_end_bcp" readonly="l10n_ro_edi_stock_fields_readonly" invisible="l10n_ro_edi_stock_end_loc_type != 'bcp'"/>
                         <field name="l10n_ro_edi_stock_end_customs_office" readonly="l10n_ro_edi_stock_fields_readonly" invisible="l10n_ro_edi_stock_end_loc_type != 'customs'"/>
                     </group>
                 </group>
             </page>

             <page id="l10n_ro_edi_stock_documents"
                   name="etransport_documents"
                   string="eTransport Documents"
                   invisible="not (l10n_ro_edi_stock_enable and l10n_ro_edi_stock_document_ids)">
                 <field name="l10n_ro_edi_stock_document_ids">
                     <list create="false" delete="false" edit="false" no_open="1"
                           decoration-danger="state == 'stock_sending_failed'"
                           decoration-warning="state == 'stock_sent'"
                           decoration-success="state == 'stock_validated'">
                         <field name="message" column_invisible="1"/>
                         <field name="attachment_id" column_invisible="1"/>
                         <field name="datetime"/>
                         <field name="state" string="Status" widget="l10n_ro_edi_stock_document_state"/>
                         <field name="l10n_ro_edi_stock_uit" string="UIT"/>
                         <field name="l10n_ro_edi_stock_load_id" string="Load Id"/>
                     </list>
                 </field>
             </page>
         </xpath>
      </field>
   </record>

    <record id="l10n_ro_edi_stock_stock_picking_batch_view_tree" model="ir.ui.view">
        <field name="model">stock.picking.batch</field>
        <field name="inherit_id" ref="stock_picking_batch.stock_picking_batch_tree"/>
        <field name="arch" type="xml">
            <field name="company_id" position="before">
                <header>
                    <button name="action_l10n_ro_edi_stock_fetch_status" string="Fetch Status" type="object"/>
                </header>
            </field>
            <field name="state" position="before">
                <field name="l10n_ro_edi_stock_state" optional="hide"/>
            </field>
        </field>
    </record>

    <record id="l10n_ro_edi_stock_stock_picking_batch_filter" model="ir.ui.view">
        <field name="model">stock.picking.batch</field>
        <field name="inherit_id" ref="stock_picking_batch.stock_picking_batch_filter"/>
        <field name="arch" type="xml">
            <field name="user_id" position="after">
                <field name="l10n_ro_edi_stock_state"/>
            </field>

            <filter name="done" position="after">
                <filter string="eTransport Error" name="l10n_ro_edi_stock_state_stock_sending_failed"
                        domain="[('l10n_ro_edi_stock_state', '=', 'stock_sending_failed')]"/>
                <filter string="eTransport Sent" name="l10n_ro_edi_stock_state_stock_sent"
                        domain="[('l10n_ro_edi_stock_state', '=', 'stock_sent')]"/>
                <filter string="eTransport Validated" name="l10n_ro_edi_stock_state_stock_validated"
                        domain="[('l10n_ro_edi_stock_state', '=', 'stock_validated')]"/>
            </filter>

            <xpath expr="//group/filter[@name='state']" position="after">
                <filter string="eTransport Status"
                        name="l10n_ro_edi_stock_state_group"
                        domain=""
                        context="{'group_by': 'l10n_ro_edi_stock_state'}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

