# Odoo Module: hr_fleet

Category: Human Resources

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Fleet History',
    'version': '1.0',
    'category': 'Human Resources',
    'summary': 'Get history of driven cars by employees',
    'description': "",
    'depends': ['hr', 'fleet'],
    'data': [
        'views/employee_views.xml',
        'views/fleet_vehicle_views.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import io

from PyPDF2 import  PdfFileReader, PdfFileWriter
from reportlab.pdfgen import canvas

from odoo import _
from odoo.http import request, route, Controller



class HrFleet(Controller):
    @route(["/fleet/print_claim_report/<int:employee_id>"], type='http', auth='user')
    def get_claim_report_user(self, employee_id, **post):
        if not request.env.user.has_group('fleet.fleet_group_manager'):
            return request.not_found()

        employee = request.env['hr.employee'].search([('id', '=', employee_id)], limit=1)
        partner_ids = (employee.user_id.partner_id | employee.sudo().address_home_id).ids
        if not employee or not partner_ids:
            return request.not_found()

        car_assignation_logs = request.env['fleet.vehicle.assignation.log'].search([('driver_id', 'in', partner_ids)])
        doc_list = request.env['ir.attachment'].search([
            ('res_model', '=', 'fleet.vehicle.assignation.log'),
            ('res_id', 'in', car_assignation_logs.ids)], order='create_date')

        writer = PdfFileWriter()

        font = "Helvetica"
        normal_font_size = 14

        for document in doc_list:
            car_line_doc = request.env['fleet.vehicle.assignation.log'].browse(document.res_id)
            try:
                reader = PdfFileReader(io.BytesIO(base64.b64decode(document.datas)), strict=False, overwriteWarnings=False)
            except Exception:
                continue

            width = float(reader.getPage(0).mediaBox.getUpperRight_x())
            height = float(reader.getPage(0).mediaBox.getUpperRight_y())

            header = io.BytesIO()
            can = canvas.Canvas(header)
            can.setFont(font, normal_font_size)
            can.setFillColorRGB(1, 0, 0)

            car_name = car_line_doc.vehicle_id.display_name
            date_start = car_line_doc.date_start
            date_end = car_line_doc.date_end or '...'

            text_to_print = _("%s (driven from: %s to %s)") % (car_name, date_start, date_end)
            can.drawCentredString(width / 2, height - normal_font_size, text_to_print)
            can.save()
            header_pdf = PdfFileReader(header, overwriteWarnings=False)

            for page_number in range(0, reader.getNumPages()):
                page = reader.getPage(page_number)
                page.mergePage(header_pdf.getPage(0))
                writer.addPage(page)

        _buffer = io.BytesIO()
        writer.write(_buffer)
        merged_pdf = _buffer.getvalue()
        _buffer.close()

        pdfhttpheaders = [('Content-Type', 'application/pdf'), ('Content-Length', len(merged_pdf))]

        return request.make_response(merged_pdf, headers=pdfhttpheaders)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Employee(models.Model):
    _inherit = 'hr.employee'

    employee_cars_count = fields.Integer(compute="_compute_employee_cars_count", string="Cars", groups="fleet.fleet_group_manager")

    def action_open_employee_cars(self):
        self.ensure_one()
        cars = self.env['fleet.vehicle.assignation.log'].search([
            ('driver_id', 'in', (self.user_id.partner_id | self.sudo().address_home_id).ids)]).mapped('vehicle_id')

        return {
            "type": "ir.actions.act_window",
            "res_model": "fleet.vehicle",
            "views": [[False, "kanban"], [False, "form"], [False, "tree"]],
            "domain": [("id", "in", cars.ids)],
            "context": dict(self._context, create=False),
            "name": "History Employee Cars",
        }

    def _compute_employee_cars_count(self):
        driver_ids = (self.mapped('user_id.partner_id') | self.sudo().mapped('address_home_id')).ids
        fleet_data = self.env['fleet.vehicle.assignation.log'].read_group(
            domain=[('driver_id', 'in', driver_ids)], fields=['vehicle_id:array_agg'], groupby=['driver_id'])
        mapped_data = {
            group['driver_id'][0]: len(set(group['vehicle_id']))
            for group in fleet_data
        }
        for employee in self:
            drivers = employee.user_id.partner_id | employee.sudo().address_home_id
            employee.employee_cars_count = sum(mapped_data.get(pid, 0) for pid in drivers.ids)

    def action_get_claim_report(self):
        self.ensure_one()
        return {
            'name': 'Claim Report',
            'type': 'ir.actions.act_url',
            'url': '/fleet/print_claim_report/%(employee_id)s' % {'employee_id': self.id},
        }

```

## File: models\fleet_vehicle_assignation_log.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class FleetVehicleAssignationLog(models.Model):
    _inherit = 'fleet.vehicle.assignation.log'

    attachment_number = fields.Integer('Number of Attachments', compute='_compute_attachment_number')

    def _compute_attachment_number(self):
        attachment_data = self.env['ir.attachment'].read_group([
            ('res_model', '=', 'fleet.vehicle.assignation.log'),
            ('res_id', 'in', self.ids)], ['res_id'], ['res_id'])
        attachment = dict((data['res_id'], data['res_id_count']) for data in attachment_data)
        for doc in self:
            doc.attachment_number = attachment.get(doc.id, 0)

    def action_get_attachment_view(self):
        self.ensure_one()
        res = self.env['ir.actions.act_window'].for_xml_id('base', 'action_attachment')
        res['domain'] = [('res_model', '=', 'fleet.vehicle.assignation.log'), ('res_id', 'in', self.ids)]
        res['context'] = {'default_res_model': 'fleet.vehicle.assignation.log', 'default_res_id': self.id}
        return res

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class User(models.Model):
    _inherit = ['res.users']

    employee_cars_count = fields.Integer(related='employee_id.employee_cars_count')

    def __init__(self, pool, cr):
        """ Override of __init__ to add access rights.
            Access rights are disabled by default, but allowed
            on some specific fields defined in self.SELF_{READ/WRITE}ABLE_FIELDS.
        """
        init_res = super(User, self).__init__(pool, cr)
        # duplicate list to avoid modifying the original reference
        type(self).SELF_READABLE_FIELDS = type(self).SELF_READABLE_FIELDS + ['employee_cars_count']
        return init_res

    def action_get_claim_report(self):
        return self.employee_id.action_get_claim_report()

    def action_open_employee_cars(self):
        return self.employee_id.action_open_employee_cars()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import employee
from . import res_users
from . import fleet_vehicle_assignation_log

```

## File: views\employee_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_employee_form" model="ir.ui.view">
        <field name="name">hr.employee.form.inherit.hr.fleet</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_form" />
        <field name="arch" type="xml">
             <xpath expr="//header" position="inside">
                <button name="action_get_claim_report" string="Claim Car Report"
                class="btn btn-secondary"
                type="object" groups="fleet.fleet_group_manager"
                attrs="{'invisible': [('employee_cars_count','=', 0)]}"
                confirm="This report will contain only PDF files. If you want all documents, please go on vehicle page. Do you want to proceed?"/>
            </xpath>
            <div name="button_box" position="inside">
                <button name="action_open_employee_cars" type="object"
                        class="oe_stat_button" icon="fa-car" groups="fleet.fleet_group_manager"
                        attrs="{'invisible': [('employee_cars_count','=', 0)]}">
                    <field name="employee_cars_count" widget="statinfo" />
                </button>
            </div>
        </field>
    </record>

    <record id="res_users_view_form_preferences" model="ir.ui.view">
        <field name="name">hr.user.preferences.form.inherit.hr.fleet</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="hr.res_users_view_form_profile" />
        <field name="arch" type="xml">
             <xpath expr="//header" position="inside">
                <button name="action_get_claim_report" string="Claim Car Report"
                class="btn btn-primary"
                type="object" groups="fleet.fleet_group_manager"
                attrs="{'invisible': [('employee_cars_count','=', 0)]}"
                confirm="This report will contain only PDF files. If you want all documents, please go on vehicle page. Do you want to proceed?"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <button name="action_open_employee_cars" type="object"
                        class="oe_stat_button" icon="fa-car" groups="fleet.fleet_group_manager"
                        attrs="{'invisible': [('employee_cars_count','=', 0)]}">
                    <field name="employee_cars_count" widget="statinfo" />
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\fleet_vehicle_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="fleet_vehicle_assignation_log_view_list" model="ir.ui.view">
        <field name="name">fleet.vehicle.assignation.log.view.tree.inherit.hr.fleet</field>
        <field name="model">fleet.vehicle.assignation.log</field>
        <field name="inherit_id" ref="fleet.fleet_vehicle_assignation_log_view_list" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='date_end']" position="after">
                <field name="attachment_number" string=" "/>
                <button name="action_get_attachment_view" string="View Attachments" type="object" icon="fa-paperclip"/>
            </xpath>
        </field>
    </record>
</odoo>

```

