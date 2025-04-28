# Odoo Module: spreadsheet_dashboard_event_sale

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': "Spreadsheet dashboard for events",
    'version': '1.0',
    'category': 'Hidden',
    'summary': 'Spreadsheet',
    'description': 'Spreadsheet',
    'depends': ['spreadsheet_dashboard', 'event_sale'],
    'data': [
        "data/dashboards.xml",
    ],
    'installable': True,
    'auto_install': ['event_sale'],
    'license': 'LGPL-3',
}

```

## File: data\dashboards.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="spreadsheet_dashboard_events" model="spreadsheet.dashboard">
        <field name="name">Events</field>
        <field name="spreadsheet_binary_data" type="base64" file="spreadsheet_dashboard_event_sale/data/files/events_dashboard.json"/>
        <field name="main_data_model_ids" eval="[(4, ref('event.model_event_event'))]"/>
        <field name="sample_dashboard_file_path">spreadsheet_dashboard_event_sale/data/files/events_sample_dashboard.json</field>
        <field name="dashboard_group_id" ref="spreadsheet_dashboard.spreadsheet_dashboard_group_marketing"/>
        <field name="group_ids" eval="[Command.link(ref('event.group_event_manager'))]"/>
        <field name="sequence">60</field>
        <field name="is_published">True</field>
    </record>

</odoo>

```

## File: data\files\events_dashboard.json

```json
{
  "version": 21,
  "sheets": [
    {
      "id": "sheet1",
      "name": "Dashboard",
      "colNumber": 5,
      "rowNumber": 56,
      "rows": {
        "6": { "size": 40 },
        "22": { "size": 40 },
        "23": { "size": 40 },
        "24": { "size": 29 },
        "25": { "size": 29 },
        "26": { "size": 29 },
        "27": { "size": 29 },
        "28": { "size": 29 },
        "29": { "size": 29 },
        "30": { "size": 29 },
        "31": { "size": 29 },
        "32": { "size": 29 },
        "33": { "size": 29 },
        "35": { "size": 40 },
        "36": { "size": 40 },
        "37": { "size": 29 },
        "38": { "size": 29 },
        "39": { "size": 29 },
        "40": { "size": 29 },
        "41": { "size": 29 },
        "42": { "size": 29 },
        "43": { "size": 29 },
        "44": { "size": 29 },
        "45": { "size": 29 },
        "46": { "size": 29 }
      },
      "cols": {
        "0": { "size": 374 },
        "1": { "size": 100 },
        "2": { "size": 50 },
        "3": { "size": 375 },
        "4": { "size": 100 }
      },
      "merges": [],
      "cells": {
        "A7": {
          "content": "[Events Status](odoo://view/{\"viewType\":\"graph\",\"action\":{\"domain\":[],\"context\":{\"group_by\":[\"stage_id\"],\"graph_measure\":\"__count\",\"graph_mode\":\"bar\",\"graph_groupbys\":[\"stage_id\"]},\"modelName\":\"event.event\",\"views\":[[false,\"kanban\"],[false,\"calendar\"],[false,\"list\"],[false,\"form\"],[false,\"pivot\"],[false,\"graph\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Events\"})"
        },
        "A23": {
          "content": "[Top Venues](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"address_id\",\"!=\",false]],\"context\":{\"group_by\":[\"address_id\"],\"pivot_measures\":[\"__count\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"address_id\"]},\"modelName\":\"event.event\",\"views\":[[false,\"kanban\"],[false,\"calendar\"],[false,\"list\"],[false,\"form\"],[false,\"pivot\"],[false,\"graph\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Events\"})"
        },
        "A24": { "content": "=_t(\"Venue\")" },
        "A25": { "content": "=PIVOT.HEADER(1,\"#address_id\",1)" },
        "A26": { "content": "=PIVOT.HEADER(1,\"#address_id\",2)" },
        "A27": { "content": "=PIVOT.HEADER(1,\"#address_id\",3)" },
        "A28": { "content": "=PIVOT.HEADER(1,\"#address_id\",4)" },
        "A29": { "content": "=PIVOT.HEADER(1,\"#address_id\",5)" },
        "A30": { "content": "=PIVOT.HEADER(1,\"#address_id\",6)" },
        "A31": { "content": "=PIVOT.HEADER(1,\"#address_id\",7)" },
        "A32": { "content": "=PIVOT.HEADER(1,\"#address_id\",8)" },
        "A33": { "content": "=PIVOT.HEADER(1,\"#address_id\",9)" },
        "A34": { "content": "=PIVOT.HEADER(1,\"#address_id\",10)" },
        "A36": {
          "content": "[Top Tags](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"tag_ids\",\"!=\",false]],\"context\":{\"group_by\":[\"tag_ids\"],\"pivot_measures\":[\"__count\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"tag_ids\"]},\"modelName\":\"event.event\",\"views\":[[false,\"kanban\"],[false,\"calendar\"],[false,\"list\"],[false,\"form\"],[false,\"pivot\"],[false,\"graph\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Events\"})"
        },
        "A37": { "content": "=_t(\"Tag\")" },
        "A38": { "content": "=PIVOT.HEADER(3,\"#tag_ids\",1)" },
        "A39": { "content": "=PIVOT.HEADER(3,\"#tag_ids\",2)" },
        "A40": { "content": "=PIVOT.HEADER(3,\"#tag_ids\",3)" },
        "A41": { "content": "=PIVOT.HEADER(3,\"#tag_ids\",4)" },
        "A42": { "content": "=PIVOT.HEADER(3,\"#tag_ids\",5)" },
        "A43": { "content": "=PIVOT.HEADER(3,\"#tag_ids\",6)" },
        "A44": { "content": "=PIVOT.HEADER(3,\"#tag_ids\",7)" },
        "A45": { "content": "=PIVOT.HEADER(3,\"#tag_ids\",8)" },
        "A46": { "content": "=PIVOT.HEADER(3,\"#tag_ids\",9)" },
        "A47": { "content": "=PIVOT.HEADER(3,\"#tag_ids\",10)" },
        "B24": { "content": "=_t(\"Events\")" },
        "B25": { "content": "=PIVOT.VALUE(1,\"__count\",\"#address_id\",1)" },
        "B26": { "content": "=PIVOT.VALUE(1,\"__count\",\"#address_id\",2)" },
        "B27": { "content": "=PIVOT.VALUE(1,\"__count\",\"#address_id\",3)" },
        "B28": { "content": "=PIVOT.VALUE(1,\"__count\",\"#address_id\",4)" },
        "B29": { "content": "=PIVOT.VALUE(1,\"__count\",\"#address_id\",5)" },
        "B30": { "content": "=PIVOT.VALUE(1,\"__count\",\"#address_id\",6)" },
        "B31": { "content": "=PIVOT.VALUE(1,\"__count\",\"#address_id\",7)" },
        "B32": { "content": "=PIVOT.VALUE(1,\"__count\",\"#address_id\",8)" },
        "B33": { "content": "=PIVOT.VALUE(1,\"__count\",\"#address_id\",9)" },
        "B34": { "content": "=PIVOT.VALUE(1,\"__count\",\"#address_id\",10)" },
        "B37": { "content": "=_t(\"Events\")" },
        "B38": { "content": "=PIVOT.VALUE(3,\"__count\",\"#tag_ids\",1)" },
        "B39": { "content": "=PIVOT.VALUE(3,\"__count\",\"#tag_ids\",2)" },
        "B40": { "content": "=PIVOT.VALUE(3,\"__count\",\"#tag_ids\",3)" },
        "B41": { "content": "=PIVOT.VALUE(3,\"__count\",\"#tag_ids\",4)" },
        "B42": { "content": "=PIVOT.VALUE(3,\"__count\",\"#tag_ids\",5)" },
        "B43": { "content": "=PIVOT.VALUE(3,\"__count\",\"#tag_ids\",6)" },
        "B44": { "content": "=PIVOT.VALUE(3,\"__count\",\"#tag_ids\",7)" },
        "B45": { "content": "=PIVOT.VALUE(3,\"__count\",\"#tag_ids\",8)" },
        "B46": { "content": "=PIVOT.VALUE(3,\"__count\",\"#tag_ids\",9)" },
        "B47": { "content": "=PIVOT.VALUE(3,\"__count\",\"#tag_ids\",10)" },
        "D7": {
          "content": "[Registration Status](odoo://view/{\"viewType\":\"graph\",\"action\":{\"domain\":[],\"context\":{\"group_by\":[\"state\"],\"graph_measure\":\"__count\",\"graph_mode\":\"bar\",\"graph_groupbys\":[\"state\"]},\"modelName\":\"event.registration\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"kanban\"],[false,\"list\"],[false,\"form\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Attendees\"})"
        },
        "D23": {
          "content": "[Top Templates](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"event_type_id\",\"!=\",false]],\"context\":{\"group_by\":[\"event_type_id\"],\"pivot_measures\":[\"__count\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"event_type_id\"]},\"modelName\":\"event.event\",\"views\":[[false,\"kanban\"],[false,\"calendar\"],[false,\"list\"],[false,\"form\"],[false,\"pivot\"],[false,\"graph\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Events\"})"
        },
        "D24": { "content": "=_t(\"Template\")" },
        "D25": { "content": "=PIVOT.HEADER(2,\"#event_type_id\",1)" },
        "D26": { "content": "=PIVOT.HEADER(2,\"#event_type_id\",2)" },
        "D27": { "content": "=PIVOT.HEADER(2,\"#event_type_id\",3)" },
        "D28": { "content": "=PIVOT.HEADER(2,\"#event_type_id\",4)" },
        "D29": { "content": "=PIVOT.HEADER(2,\"#event_type_id\",5)" },
        "D30": { "content": "=PIVOT.HEADER(2,\"#event_type_id\",6)" },
        "D31": { "content": "=PIVOT.HEADER(2,\"#event_type_id\",7)" },
        "D32": { "content": "=PIVOT.HEADER(2,\"#event_type_id\",8)" },
        "D33": { "content": "=PIVOT.HEADER(2,\"#event_type_id\",9)" },
        "D34": { "content": "=PIVOT.HEADER(2,\"#event_type_id\",10)" },
        "D36": {
          "content": "[Top Organizers](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"organizer_id\",\"!=\",false]],\"context\":{\"group_by\":[\"organizer_id\"],\"pivot_measures\":[\"__count\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"organizer_id\"]},\"modelName\":\"event.event\",\"views\":[[false,\"kanban\"],[false,\"calendar\"],[false,\"list\"],[false,\"form\"],[false,\"pivot\"],[false,\"graph\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Events\"})"
        },
        "D37": { "content": "=_t(\"Organizer\")" },
        "D38": { "content": "=PIVOT.HEADER(4,\"#organizer_id\",1)" },
        "D39": { "content": "=PIVOT.HEADER(4,\"#organizer_id\",2)" },
        "D40": { "content": "=PIVOT.HEADER(4,\"#organizer_id\",3)" },
        "D41": { "content": "=PIVOT.HEADER(4,\"#organizer_id\",4)" },
        "D42": { "content": "=PIVOT.HEADER(4,\"#organizer_id\",5)" },
        "D43": { "content": "=PIVOT.HEADER(4,\"#organizer_id\",6)" },
        "D44": { "content": "=PIVOT.HEADER(4,\"#organizer_id\",7)" },
        "D45": { "content": "=PIVOT.HEADER(4,\"#organizer_id\",8)" },
        "D46": { "content": "=PIVOT.HEADER(4,\"#organizer_id\",9)" },
        "D47": { "content": "=PIVOT.HEADER(4,\"#organizer_id\",10)" },
        "E24": { "content": "=_t(\"Events\")" },
        "E25": {
          "content": "=PIVOT.VALUE(2,\"__count\",\"#event_type_id\",1)"
        },
        "E26": {
          "content": "=PIVOT.VALUE(2,\"__count\",\"#event_type_id\",2)"
        },
        "E27": {
          "content": "=PIVOT.VALUE(2,\"__count\",\"#event_type_id\",3)"
        },
        "E28": {
          "content": "=PIVOT.VALUE(2,\"__count\",\"#event_type_id\",4)"
        },
        "E29": {
          "content": "=PIVOT.VALUE(2,\"__count\",\"#event_type_id\",5)"
        },
        "E30": {
          "content": "=PIVOT.VALUE(2,\"__count\",\"#event_type_id\",6)"
        },
        "E31": {
          "content": "=PIVOT.VALUE(2,\"__count\",\"#event_type_id\",7)"
        },
        "E32": {
          "content": "=PIVOT.VALUE(2,\"__count\",\"#event_type_id\",8)"
        },
        "E33": {
          "content": "=PIVOT.VALUE(2,\"__count\",\"#event_type_id\",9)"
        },
        "E34": {
          "content": "=PIVOT.VALUE(2,\"__count\",\"#event_type_id\",10)"
        },
        "E37": { "content": "=_t(\"Events\")" },
        "E38": { "content": "=PIVOT.VALUE(4,\"__count\",\"#organizer_id\",1)" },
        "E39": { "content": "=PIVOT.VALUE(4,\"__count\",\"#organizer_id\",2)" },
        "E40": { "content": "=PIVOT.VALUE(4,\"__count\",\"#organizer_id\",3)" },
        "E41": { "content": "=PIVOT.VALUE(4,\"__count\",\"#organizer_id\",4)" },
        "E42": { "content": "=PIVOT.VALUE(4,\"__count\",\"#organizer_id\",5)" },
        "E43": { "content": "=PIVOT.VALUE(4,\"__count\",\"#organizer_id\",6)" },
        "E44": { "content": "=PIVOT.VALUE(4,\"__count\",\"#organizer_id\",7)" },
        "E45": { "content": "=PIVOT.VALUE(4,\"__count\",\"#organizer_id\",8)" },
        "E46": { "content": "=PIVOT.VALUE(4,\"__count\",\"#organizer_id\",9)" },
        "E47": { "content": "=PIVOT.VALUE(4,\"__count\",\"#organizer_id\",10)" }
      },
      "styles": {
        "A7": 1,
        "A23": 1,
        "A36": 1,
        "D7": 1,
        "D23": 1,
        "D36": 1,
        "A24": 2,
        "A37": 2,
        "D24": 2,
        "D37": 2,
        "A25:B34": 3,
        "A38:B47": 3,
        "D25:E34": 3,
        "D38:E47": 3,
        "B24": 4,
        "B37": 4,
        "E24": 4,
        "E37": 4
      },
      "formats": {},
      "borders": {
        "A7:B7": 1,
        "A23:B23": 1,
        "A36:B36": 1,
        "D7:E7": 1,
        "D23:E23": 1,
        "D36:E36": 1,
        "A8:B8": 2,
        "A24:B24": 2,
        "A37:B37": 2,
        "D8:E8": 2,
        "D24:E24": 2,
        "D37:E37": 2,
        "A25": 3,
        "A38": 3,
        "D25": 3,
        "D38": 3,
        "A26:A34": 4,
        "A39:A47": 4,
        "D26:D34": 4,
        "D39:D47": 4,
        "A35:B35": 5,
        "A48:B48": 5,
        "D35:E35": 5,
        "D48:E48": 5,
        "B25": 6,
        "B38": 6,
        "E25": 6,
        "E38": 6,
        "B26:B34": 7,
        "B39:B47": 7,
        "E26:E34": 7,
        "E39:E47": 7
      },
      "conditionalFormats": [
        {
          "rule": {
            "type": "DataBarRule",
            "color": 15726335,
            "rangeValues": "B25:B34"
          },
          "id": "813e8f10-5d2d-42bf-994d-84a21425573c",
          "ranges": ["A25:A34"]
        },
        {
          "rule": {
            "type": "DataBarRule",
            "color": 16775149,
            "rangeValues": "E25:E34"
          },
          "id": "95cfb086-980c-418f-b6cf-65663c0fe449",
          "ranges": ["D25:D34"]
        },
        {
          "rule": {
            "type": "DataBarRule",
            "color": 15531509,
            "rangeValues": "B38:B47"
          },
          "id": "3a61df0c-f147-4d90-a6a7-08e4b50e495a",
          "ranges": ["A38:A47"]
        },
        {
          "rule": {
            "type": "DataBarRule",
            "color": 16708338,
            "rangeValues": "E38:E47"
          },
          "id": "0b11b334-4302-4e29-b4e7-e2d964d6c999",
          "ranges": ["D38:D47"]
        }
      ],
      "figures": [
        {
          "id": "d9898682-9b91-4fe2-9900-a4b2583eb459",
          "x": 525,
          "y": 178,
          "width": 475,
          "height": 345,
          "tag": "chart",
          "data": {
            "title": { "text": "" },
            "background": "#FFFFFF",
            "legendPosition": "none",
            "metaData": {
              "groupBy": ["state"],
              "measure": "__count",
              "order": null,
              "resModel": "event.registration",
              "mode": "bar"
            },
            "searchParams": {
              "comparison": null,
              "context": {
                "registration_view_hide_group_by_create_date_week": 1
              },
              "domain": [],
              "groupBy": ["state"],
              "orderBy": []
            },
            "type": "odoo_bar",
            "verticalAxisPosition": "left",
            "stacked": true,
            "fieldMatching": {
              "f849aaa6-4c1a-43f6-9c0b-7a5ea9c83eae": {
                "chain": "event_begin_date",
                "type": "datetime",
                "offset": 0
              },
              "62e15eb2-9410-4803-b189-5fa789a7e94f": {
                "chain": "event_id.address_id",
                "type": "many2one"
              },
              "d9cfb0b0-01a0-45be-996e-723f4283000b": {
                "chain": "event_id.event_type_id",
                "type": "many2one"
              },
              "43c04fb0-d3b7-47c8-852b-b823c36df0ee": {
                "chain": "event_id.tag_ids",
                "type": "many2many"
              },
              "4a35bef8-8a57-48ae-8dc6-5883d97cc9da": {
                "chain": "event_organizer_id",
                "type": "many2one"
              }
            }
          }
        },
        {
          "id": "092dd971-dc10-41c5-96c4-d760820e9cf2",
          "x": 0,
          "y": 178,
          "width": 475,
          "height": 345,
          "tag": "chart",
          "data": {
            "title": { "text": "" },
            "background": "#FFFFFF",
            "legendPosition": "none",
            "metaData": {
              "groupBy": ["stage_id"],
              "measure": "__count",
              "order": null,
              "resModel": "event.event",
              "mode": "bar"
            },
            "searchParams": {
              "comparison": null,
              "context": {},
              "domain": [],
              "groupBy": ["stage_id"],
              "orderBy": []
            },
            "type": "odoo_bar",
            "verticalAxisPosition": "left",
            "stacked": true,
            "fieldMatching": {
              "f849aaa6-4c1a-43f6-9c0b-7a5ea9c83eae": {
                "chain": "date_begin",
                "type": "datetime",
                "offset": 0
              },
              "62e15eb2-9410-4803-b189-5fa789a7e94f": {
                "chain": "address_id",
                "type": "many2one"
              },
              "d9cfb0b0-01a0-45be-996e-723f4283000b": {
                "chain": "event_type_id",
                "type": "many2one"
              },
              "43c04fb0-d3b7-47c8-852b-b823c36df0ee": {
                "chain": "tag_ids",
                "type": "many2many"
              },
              "4a35bef8-8a57-48ae-8dc6-5883d97cc9da": {
                "chain": "organizer_id",
                "type": "many2one"
              }
            }
          }
        },
        {
          "id": "18361546-49c2-4212-88eb-b2ee50127d41",
          "x": 0,
          "y": 9,
          "width": 200,
          "height": 105,
          "tag": "chart",
          "data": {
            "baselineColorDown": "#DC6965",
            "baselineColorUp": "#00A04A",
            "baselineMode": "percentage",
            "title": { "text": "Events", "bold": true, "color": "#434343" },
            "type": "scorecard",
            "background": "#EFF6FF",
            "baseline": "Data!E3",
            "baselineDescr": "since last period",
            "keyValue": "Data!D3",
            "humanize": false
          }
        },
        {
          "id": "98d2b15d-9117-4ecf-9732-dc62df949dc1",
          "x": 210,
          "y": 9,
          "width": 200,
          "height": 105,
          "tag": "chart",
          "data": {
            "baselineColorDown": "#DC6965",
            "baselineColorUp": "#00A04A",
            "baselineMode": "percentage",
            "title": { "text": "Revenue", "bold": true, "color": "#434343" },
            "type": "scorecard",
            "background": "#FFF7ED",
            "baseline": "Data!E4",
            "baselineDescr": "since last period",
            "keyValue": "Data!D4",
            "humanize": false
          }
        },
        {
          "id": "6510bff8-7b28-42c6-af4b-c750bed2205c",
          "x": 420,
          "y": 9,
          "width": 200,
          "height": 105,
          "tag": "chart",
          "data": {
            "baselineColorDown": "#DC6965",
            "baselineColorUp": "#00A04A",
            "baselineMode": "percentage",
            "title": { "text": "Attendees", "bold": true, "color": "#434343" },
            "type": "scorecard",
            "background": "#EFF6FF",
            "baseline": "Data!E2",
            "baselineDescr": "since last period",
            "keyValue": "Data!D2",
            "humanize": false
          }
        }
      ],
      "tables": [],
      "areGridLinesVisible": true,
      "isVisible": true,
      "headerGroups": { "ROW": [], "COL": [] },
      "dataValidationRules": [],
      "comments": {}
    },
    {
      "id": "3fad37db-0a40-46e9-bcc8-967eebf0bca4",
      "name": "Data",
      "colNumber": 26,
      "rowNumber": 101,
      "rows": {},
      "cols": {
        "1": { "size": 93 },
        "2": { "size": 93 },
        "3": { "size": 93 },
        "4": { "size": 93 }
      },
      "merges": [],
      "cells": {
        "A1": { "content": "=_t(\"KPI\")" },
        "A2": { "content": "=_t(\"Attendees\")" },
        "A3": { "content": "=_t(\"Events\")" },
        "A4": { "content": "=_t(\"Revenue\")" },
        "B1": { "content": "=_t(\"Current\")" },
        "B2": { "content": "=PIVOT.VALUE(5,\"__count\")" },
        "B3": { "content": "=PIVOT.VALUE(9,\"__count\")" },
        "B4": { "content": "=PIVOT.VALUE(7,\"sale_price_untaxed\")" },
        "C1": { "content": "=_t(\"Previous\")" },
        "C2": { "content": "=PIVOT.VALUE(6,\"__count\")" },
        "C3": { "content": "=PIVOT.VALUE(10,\"__count\")" },
        "C4": { "content": "=PIVOT.VALUE(8,\"sale_price_untaxed\")" },
        "D1": { "content": "=_t(\"Current\")" },
        "D2": { "content": "=FORMAT.LARGE.NUMBER(B2)" },
        "D3": { "content": "=FORMAT.LARGE.NUMBER(B3)" },
        "D4": { "content": "=FORMAT.LARGE.NUMBER(B4)" },
        "E1": { "content": "=_t(\"Previous\")" },
        "E2": { "content": "=FORMAT.LARGE.NUMBER(C2)" },
        "E3": { "content": "=FORMAT.LARGE.NUMBER(C3)" },
        "E4": { "content": "=FORMAT.LARGE.NUMBER(C4)" }
      },
      "styles": { "A1:C1": 5, "D1:E1": 6, "D2:E4": 7 },
      "formats": {},
      "borders": {},
      "conditionalFormats": [],
      "figures": [],
      "tables": [],
      "areGridLinesVisible": true,
      "isVisible": true,
      "headerGroups": { "ROW": [], "COL": [] },
      "dataValidationRules": [],
      "comments": {}
    }
  ],
  "styles": {
    "1": { "textColor": "#01666b", "fontSize": 16, "bold": true },
    "2": { "textColor": "#434343", "bold": true, "fontSize": 11 },
    "3": { "textColor": "#434343", "verticalAlign": "middle" },
    "4": {
      "textColor": "#434343",
      "bold": true,
      "fontSize": 11,
      "align": "center"
    },
    "5": { "bold": true },
    "6": { "bold": true, "fillColor": "#f2f2f2" },
    "7": { "fillColor": "#f2f2f2" }
  },
  "formats": {},
  "borders": {
    "1": { "bottom": { "style": "thin", "color": "#CCCCCC" } },
    "2": { "top": { "style": "thin", "color": "#CCCCCC" } },
    "3": {
      "bottom": { "style": "thick", "color": "#FFFFFF" },
      "right": { "style": "thick", "color": "#FFFFFF" }
    },
    "4": {
      "top": { "style": "thick", "color": "#FFFFFF" },
      "bottom": { "style": "thick", "color": "#FFFFFF" },
      "right": { "style": "thick", "color": "#FFFFFF" }
    },
    "5": { "top": { "style": "thick", "color": "#FFFFFF" } },
    "6": {
      "bottom": { "style": "thick", "color": "#FFFFFF" },
      "left": { "style": "thick", "color": "#FFFFFF" }
    },
    "7": {
      "top": { "style": "thick", "color": "#FFFFFF" },
      "bottom": { "style": "thick", "color": "#FFFFFF" },
      "left": { "style": "thick", "color": "#FFFFFF" }
    }
  },
  "revisionId": "START_REVISION",
  "uniqueFigureIds": true,
  "settings": {
    "locale": {
      "name": "English (US)",
      "code": "en_US",
      "thousandsSeparator": ",",
      "decimalSeparator": ".",
      "dateFormat": "mm/dd/yyyy",
      "timeFormat": "hh:mm:ss",
      "formulaArgSeparator": ",",
      "weekStart": 7
    }
  },
  "pivots": {
    "1": {
      "type": "ODOO",
      "fieldMatching": {
        "f849aaa6-4c1a-43f6-9c0b-7a5ea9c83eae": {
          "chain": "date_begin",
          "type": "datetime",
          "offset": 0
        },
        "62e15eb2-9410-4803-b189-5fa789a7e94f": {
          "chain": "address_id",
          "type": "many2one"
        },
        "d9cfb0b0-01a0-45be-996e-723f4283000b": {
          "chain": "event_type_id",
          "type": "many2one"
        },
        "43c04fb0-d3b7-47c8-852b-b823c36df0ee": {
          "chain": "tag_ids",
          "type": "many2many"
        },
        "4a35bef8-8a57-48ae-8dc6-5883d97cc9da": {
          "chain": "organizer_id",
          "type": "many2one"
        }
      },
      "context": {},
      "domain": [["address_id", "!=", false]],
      "id": "1",
      "measures": [{ "id": "__count", "fieldName": "__count" }],
      "model": "event.event",
      "name": "Event by Venue",
      "sortedColumn": {
        "groupId": [[], []],
        "measure": "__count",
        "order": "desc"
      },
      "formulaId": "1",
      "columns": [],
      "rows": [{ "fieldName": "address_id" }]
    },
    "2": {
      "type": "ODOO",
      "fieldMatching": {
        "f849aaa6-4c1a-43f6-9c0b-7a5ea9c83eae": {
          "chain": "date_begin",
          "type": "datetime",
          "offset": 0
        },
        "62e15eb2-9410-4803-b189-5fa789a7e94f": {
          "chain": "address_id",
          "type": "many2one"
        },
        "d9cfb0b0-01a0-45be-996e-723f4283000b": {
          "chain": "event_type_id",
          "type": "many2one"
        },
        "43c04fb0-d3b7-47c8-852b-b823c36df0ee": {
          "chain": "tag_ids",
          "type": "many2many"
        },
        "4a35bef8-8a57-48ae-8dc6-5883d97cc9da": {
          "chain": "organizer_id",
          "type": "many2one"
        }
      },
      "context": {},
      "domain": [["event_type_id", "!=", false]],
      "id": "2",
      "measures": [{ "id": "__count", "fieldName": "__count" }],
      "model": "event.event",
      "name": "Event by Template",
      "sortedColumn": {
        "groupId": [[], []],
        "measure": "__count",
        "order": "desc"
      },
      "formulaId": "2",
      "columns": [],
      "rows": [{ "fieldName": "event_type_id" }]
    },
    "3": {
      "type": "ODOO",
      "fieldMatching": {
        "f849aaa6-4c1a-43f6-9c0b-7a5ea9c83eae": {
          "chain": "date_begin",
          "type": "datetime",
          "offset": 0
        },
        "62e15eb2-9410-4803-b189-5fa789a7e94f": {
          "chain": "address_id",
          "type": "many2one"
        },
        "d9cfb0b0-01a0-45be-996e-723f4283000b": {
          "chain": "event_type_id",
          "type": "many2one"
        },
        "43c04fb0-d3b7-47c8-852b-b823c36df0ee": {
          "chain": "tag_ids",
          "type": "many2many"
        },
        "4a35bef8-8a57-48ae-8dc6-5883d97cc9da": {
          "chain": "organizer_id",
          "type": "many2one"
        }
      },
      "context": {},
      "domain": [["tag_ids", "!=", false]],
      "id": "3",
      "measures": [{ "id": "__count", "fieldName": "__count" }],
      "model": "event.event",
      "name": "Event by Tags",
      "sortedColumn": {
        "groupId": [[], []],
        "measure": "__count",
        "order": "desc"
      },
      "formulaId": "3",
      "columns": [],
      "rows": [{ "fieldName": "tag_ids" }]
    },
    "4": {
      "type": "ODOO",
      "fieldMatching": {
        "f849aaa6-4c1a-43f6-9c0b-7a5ea9c83eae": {
          "chain": "date_begin",
          "type": "datetime",
          "offset": 0
        },
        "62e15eb2-9410-4803-b189-5fa789a7e94f": {
          "chain": "address_id",
          "type": "many2one"
        },
        "d9cfb0b0-01a0-45be-996e-723f4283000b": {
          "chain": "event_type_id",
          "type": "many2one"
        },
        "43c04fb0-d3b7-47c8-852b-b823c36df0ee": {
          "chain": "tag_ids",
          "type": "many2many"
        },
        "4a35bef8-8a57-48ae-8dc6-5883d97cc9da": {
          "chain": "organizer_id",
          "type": "many2one"
        }
      },
      "context": {},
      "domain": [["organizer_id", "!=", false]],
      "id": "4",
      "measures": [{ "id": "__count", "fieldName": "__count" }],
      "model": "event.event",
      "name": "Event by Organizer",
      "sortedColumn": {
        "groupId": [[], []],
        "measure": "__count",
        "order": "desc"
      },
      "formulaId": "4",
      "columns": [],
      "rows": [{ "fieldName": "organizer_id" }]
    },
    "5": {
      "type": "ODOO",
      "fieldMatching": {
        "f849aaa6-4c1a-43f6-9c0b-7a5ea9c83eae": {
          "chain": "event_begin_date",
          "type": "datetime",
          "offset": 0
        },
        "62e15eb2-9410-4803-b189-5fa789a7e94f": {
          "chain": "event_id.address_id",
          "type": "many2one"
        },
        "d9cfb0b0-01a0-45be-996e-723f4283000b": {
          "chain": "event_id.event_type_id",
          "type": "many2one"
        },
        "43c04fb0-d3b7-47c8-852b-b823c36df0ee": {
          "chain": "event_id.tag_ids",
          "type": "many2many"
        },
        "4a35bef8-8a57-48ae-8dc6-5883d97cc9da": {
          "chain": "event_organizer_id",
          "type": "many2one"
        }
      },
      "context": { "registration_view_hide_group_by_create_date_week": 1 },
      "domain": [],
      "id": "5",
      "measures": [{ "id": "__count", "fieldName": "__count" }],
      "model": "event.registration",
      "name": "attendees - current",
      "sortedColumn": null,
      "formulaId": "5",
      "columns": [],
      "rows": []
    },
    "6": {
      "type": "ODOO",
      "fieldMatching": {
        "f849aaa6-4c1a-43f6-9c0b-7a5ea9c83eae": {
          "chain": "event_begin_date",
          "type": "datetime",
          "offset": -1
        },
        "62e15eb2-9410-4803-b189-5fa789a7e94f": {
          "chain": "event_id.address_id",
          "type": "many2one"
        },
        "d9cfb0b0-01a0-45be-996e-723f4283000b": {
          "chain": "event_id.event_type_id",
          "type": "many2one"
        },
        "43c04fb0-d3b7-47c8-852b-b823c36df0ee": {
          "chain": "event_id.tag_ids",
          "type": "many2many"
        },
        "4a35bef8-8a57-48ae-8dc6-5883d97cc9da": {
          "chain": "event_organizer_id",
          "type": "many2one"
        }
      },
      "context": { "registration_view_hide_group_by_create_date_week": 1 },
      "domain": [],
      "id": "6",
      "measures": [{ "id": "__count", "fieldName": "__count" }],
      "model": "event.registration",
      "name": "attendees - previous",
      "sortedColumn": null,
      "formulaId": "6",
      "columns": [],
      "rows": []
    },
    "7": {
      "type": "ODOO",
      "fieldMatching": {
        "f849aaa6-4c1a-43f6-9c0b-7a5ea9c83eae": {
          "chain": "event_date_begin",
          "type": "date",
          "offset": 0
        },
        "62e15eb2-9410-4803-b189-5fa789a7e94f": {
          "chain": "event_id.address_id",
          "type": "many2one"
        },
        "d9cfb0b0-01a0-45be-996e-723f4283000b": {
          "chain": "event_type_id",
          "type": "many2one"
        },
        "43c04fb0-d3b7-47c8-852b-b823c36df0ee": {
          "chain": "event_id.tag_ids",
          "type": "many2many"
        },
        "4a35bef8-8a57-48ae-8dc6-5883d97cc9da": {
          "chain": "event_id.organizer_id",
          "type": "many2one"
        }
      },
      "context": {
        "pivot_measures": ["__count__", "sale_price_untaxed", "sale_price"]
      },
      "domain": [],
      "id": "7",
      "measures": [
        { "id": "sale_price_untaxed", "fieldName": "sale_price_untaxed" }
      ],
      "model": "event.sale.report",
      "name": "untaxed revenue - current",
      "sortedColumn": null,
      "formulaId": "7",
      "columns": [],
      "rows": []
    },
    "8": {
      "type": "ODOO",
      "fieldMatching": {
        "f849aaa6-4c1a-43f6-9c0b-7a5ea9c83eae": {
          "chain": "event_date_begin",
          "type": "date",
          "offset": -1
        },
        "62e15eb2-9410-4803-b189-5fa789a7e94f": {
          "chain": "event_id.address_id",
          "type": "many2one"
        },
        "d9cfb0b0-01a0-45be-996e-723f4283000b": {
          "chain": "event_type_id",
          "type": "many2one"
        },
        "43c04fb0-d3b7-47c8-852b-b823c36df0ee": {
          "chain": "event_id.tag_ids",
          "type": "many2many"
        },
        "4a35bef8-8a57-48ae-8dc6-5883d97cc9da": {
          "chain": "event_id.organizer_id",
          "type": "many2one"
        }
      },
      "context": {
        "pivot_measures": ["__count__", "sale_price_untaxed", "sale_price"]
      },
      "domain": [],
      "id": "8",
      "measures": [
        { "id": "__count", "fieldName": "__count" },
        { "id": "sale_price_untaxed", "fieldName": "sale_price_untaxed" },
        { "id": "sale_price", "fieldName": "sale_price" }
      ],
      "model": "event.sale.report",
      "name": "untaxed revenue - previous",
      "sortedColumn": null,
      "formulaId": "8",
      "columns": [],
      "rows": []
    },
    "9": {
      "type": "ODOO",
      "fieldMatching": {
        "f849aaa6-4c1a-43f6-9c0b-7a5ea9c83eae": {
          "chain": "date_begin",
          "type": "datetime",
          "offset": 0
        },
        "62e15eb2-9410-4803-b189-5fa789a7e94f": {
          "chain": "address_id",
          "type": "many2one"
        },
        "d9cfb0b0-01a0-45be-996e-723f4283000b": {
          "chain": "event_type_id",
          "type": "many2one"
        },
        "43c04fb0-d3b7-47c8-852b-b823c36df0ee": {
          "chain": "tag_ids",
          "type": "many2many"
        },
        "4a35bef8-8a57-48ae-8dc6-5883d97cc9da": {
          "chain": "organizer_id",
          "type": "many2one"
        }
      },
      "context": {},
      "domain": [],
      "id": "9",
      "measures": [{ "id": "__count", "fieldName": "__count" }],
      "model": "event.event",
      "name": "events - current",
      "sortedColumn": null,
      "formulaId": "9",
      "columns": [],
      "rows": []
    },
    "10": {
      "type": "ODOO",
      "fieldMatching": {
        "f849aaa6-4c1a-43f6-9c0b-7a5ea9c83eae": {
          "chain": "date_begin",
          "type": "datetime",
          "offset": -1
        },
        "62e15eb2-9410-4803-b189-5fa789a7e94f": {
          "chain": "address_id",
          "type": "many2one"
        },
        "d9cfb0b0-01a0-45be-996e-723f4283000b": {
          "chain": "event_type_id",
          "type": "many2one"
        },
        "43c04fb0-d3b7-47c8-852b-b823c36df0ee": {
          "chain": "tag_ids",
          "type": "many2many"
        },
        "4a35bef8-8a57-48ae-8dc6-5883d97cc9da": {
          "chain": "organizer_id",
          "type": "many2one"
        }
      },
      "context": {},
      "domain": [],
      "id": "10",
      "measures": [{ "id": "__count", "fieldName": "__count" }],
      "model": "event.event",
      "name": "events - previous",
      "sortedColumn": null,
      "formulaId": "10",
      "columns": [],
      "rows": []
    }
  },
  "pivotNextId": 11,
  "customTableStyles": {},
  "odooVersion": 12,
  "globalFilters": [
    {
      "id": "f849aaa6-4c1a-43f6-9c0b-7a5ea9c83eae",
      "type": "date",
      "label": "Date",
      "defaultValue": "last_year",
      "rangeType": "relative"
    },
    {
      "id": "62e15eb2-9410-4803-b189-5fa789a7e94f",
      "type": "relation",
      "label": "Venue",
      "modelName": "res.partner",
      "defaultValue": [],
      "defaultValueDisplayNames": [],
      "rangeType": "year"
    },
    {
      "id": "d9cfb0b0-01a0-45be-996e-723f4283000b",
      "type": "relation",
      "label": "Template",
      "modelName": "event.type",
      "defaultValue": [],
      "defaultValueDisplayNames": [],
      "rangeType": "year"
    },
    {
      "id": "43c04fb0-d3b7-47c8-852b-b823c36df0ee",
      "type": "relation",
      "label": "Tags",
      "modelName": "event.tag",
      "defaultValue": [],
      "defaultValueDisplayNames": [],
      "rangeType": "year"
    },
    {
      "id": "4a35bef8-8a57-48ae-8dc6-5883d97cc9da",
      "type": "relation",
      "label": "Organizer",
      "modelName": "res.partner",
      "defaultValue": [],
      "defaultValueDisplayNames": [],
      "rangeType": "year"
    }
  ],
  "lists": {},
  "listNextId": 1,
  "chartOdooMenusReferences": {
    "d9898682-9b91-4fe2-9900-a4b2583eb459": "event.event_main_menu",
    "092dd971-dc10-41c5-96c4-d760820e9cf2": "event.event_main_menu",
    "18361546-49c2-4212-88eb-b2ee50127d41": "event.menu_event_event",
    "98d2b15d-9117-4ecf-9732-dc62df949dc1": "event_sale.menu_action_show_revenues",
    "6510bff8-7b28-42c6-af4b-c750bed2205c": "event.menu_action_registration"
  }
}

```

## File: data\files\events_sample_dashboard.json

```json
{
    "version": 21,
    "sheets": [
        {
            "id": "sheet1",
            "name": "Dashboard",
            "colNumber": 5,
            "rowNumber": 49,
            "rows": {
                "6": {
                    "size": 40
                },
                "22": {
                    "size": 40
                },
                "23": {
                    "size": 40
                },
                "24": {
                    "size": 29
                },
                "25": {
                    "size": 29
                },
                "26": {
                    "size": 29
                },
                "27": {
                    "size": 29
                },
                "28": {
                    "size": 29
                },
                "29": {
                    "size": 29
                },
                "30": {
                    "size": 29
                },
                "31": {
                    "size": 29
                },
                "32": {
                    "size": 29
                },
                "33": {
                    "size": 29
                },
                "35": {
                    "size": 40
                },
                "36": {
                    "size": 40
                },
                "37": {
                    "size": 29
                },
                "38": {
                    "size": 29
                },
                "39": {
                    "size": 29
                },
                "40": {
                    "size": 29
                },
                "41": {
                    "size": 29
                },
                "42": {
                    "size": 29
                },
                "43": {
                    "size": 29
                },
                "44": {
                    "size": 29
                },
                "45": {
                    "size": 29
                },
                "46": {
                    "size": 29
                }
            },
            "cols": {
                "0": {
                    "size": 374
                },
                "1": {
                    "size": 100
                },
                "2": {
                    "size": 50
                },
                "3": {
                    "size": 375
                },
                "4": {
                    "size": 100
                }
            },
            "merges": [],
            "cells": {
                "A7": {
                    "content": "[Events Status](odoo://view/{\"viewType\":\"graph\",\"action\":{\"domain\":[],\"context\":{\"group_by\":[\"stage_id\"],\"graph_measure\":\"__count\",\"graph_mode\":\"bar\",\"graph_groupbys\":[\"stage_id\"]},\"modelName\":\"event.event\",\"views\":[[false,\"kanban\"],[false,\"calendar\"],[false,\"list\"],[false,\"form\"],[false,\"pivot\"],[false,\"graph\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Events\"})"
                },
                "A23": {
                    "content": "[Top Venues](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"address_id\",\"!=\",false]],\"context\":{\"group_by\":[\"address_id\"],\"pivot_measures\":[\"__count\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"address_id\"]},\"modelName\":\"event.event\",\"views\":[[false,\"kanban\"],[false,\"calendar\"],[false,\"list\"],[false,\"form\"],[false,\"pivot\"],[false,\"graph\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Events\"})"
                },
                "A24": {
                    "content": "=_t(\"Venue\")"
                },
                "A36": {
                    "content": "[Top Tags](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"tag_ids\",\"!=\",false]],\"context\":{\"group_by\":[\"tag_ids\"],\"pivot_measures\":[\"__count\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"tag_ids\"]},\"modelName\":\"event.event\",\"views\":[[false,\"kanban\"],[false,\"calendar\"],[false,\"list\"],[false,\"form\"],[false,\"pivot\"],[false,\"graph\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Events\"})"
                },
                "A37": {
                    "content": "=_t(\"Tag\")"
                },
                "B24": {
                    "content": "=_t(\"Events\")"
                },
                "B37": {
                    "content": "=_t(\"Events\")"
                },
                "D7": {
                    "content": "[Registration Status](odoo://view/{\"viewType\":\"graph\",\"action\":{\"domain\":[],\"context\":{\"group_by\":[\"state\"],\"graph_measure\":\"__count\",\"graph_mode\":\"bar\",\"graph_groupbys\":[\"state\"]},\"modelName\":\"event.registration\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"kanban\"],[false,\"list\"],[false,\"form\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Attendees\"})"
                },
                "D23": {
                    "content": "[Top Templates](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"event_type_id\",\"!=\",false]],\"context\":{\"group_by\":[\"event_type_id\"],\"pivot_measures\":[\"__count\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"event_type_id\"]},\"modelName\":\"event.event\",\"views\":[[false,\"kanban\"],[false,\"calendar\"],[false,\"list\"],[false,\"form\"],[false,\"pivot\"],[false,\"graph\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Events\"})"
                },
                "D24": {
                    "content": "=_t(\"Template\")"
                },
                "D36": {
                    "content": "[Top Organizers](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"organizer_id\",\"!=\",false]],\"context\":{\"group_by\":[\"organizer_id\"],\"pivot_measures\":[\"__count\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"organizer_id\"]},\"modelName\":\"event.event\",\"views\":[[false,\"kanban\"],[false,\"calendar\"],[false,\"list\"],[false,\"form\"],[false,\"pivot\"],[false,\"graph\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Events\"})"
                },
                "D37": {
                    "content": "=_t(\"Organizer\")"
                },
                "E24": {
                    "content": "=_t(\"Events\")"
                },
                "E37": {
                    "content": "=_t(\"Events\")"
                }
            },
            "styles": {
                "A7": 1,
                "A23": 1,
                "A36": 1,
                "D7": 1,
                "D23": 1,
                "D36": 1,
                "A24": 2,
                "A37": 2,
                "D24": 2,
                "D37": 2,
                "B24": 3,
                "B37": 3,
                "E24": 3,
                "E37": 3
            },
            "formats": {},
            "borders": {
                "A7:B7": 1,
                "A23:B23": 1,
                "A36:B36": 1,
                "D7:E7": 1,
                "D23:E23": 1,
                "D36:E36": 1,
                "A8:B8": 2,
                "A24:B24": 2,
                "A37:B37": 2,
                "D8:E8": 2,
                "D24:E24": 2,
                "D37:E37": 2,
                "A25": 3,
                "A38": 3,
                "D25": 3,
                "D38": 3,
                "A26:A34": 4,
                "A39:A43": 4,
                "A45:A47": 4,
                "D26:D34": 4,
                "D39:D43": 4,
                "D45:D47": 4,
                "A35:B35": 5,
                "A48:B48": 5,
                "D35:E35": 5,
                "D48:E48": 5,
                "A44:B44": 6,
                "D44:E44": 6,
                "B25": 7,
                "B38": 7,
                "E25": 7,
                "E38": 7,
                "B26:B34": 8,
                "B39:B43": 8,
                "B45:B47": 8,
                "E26:E34": 8,
                "E39:E43": 8,
                "E45:E47": 8
            },
            "conditionalFormats": [],
            "figures": [
                {
                    "id": "18361546-49c2-4212-88eb-b2ee50127d41",
                    "x": 0,
                    "y": 9,
                    "width": 200,
                    "height": 105,
                    "tag": "chart",
                    "data": {
                        "baselineColorDown": "#DC6965",
                        "baselineColorUp": "#00A04A",
                        "baselineMode": "percentage",
                        "title": {
                            "text": "Events",
                            "bold": true,
                            "color": "#434343"
                        },
                        "type": "scorecard",
                        "background": "#EFF6FF",
                        "baseline": "Data!E3",
                        "baselineDescr": "since last period",
                        "keyValue": "Data!D3",
                        "humanize": false
                    }
                },
                {
                    "id": "98d2b15d-9117-4ecf-9732-dc62df949dc1",
                    "x": 210,
                    "y": 9,
                    "width": 200,
                    "height": 105,
                    "tag": "chart",
                    "data": {
                        "baselineColorDown": "#DC6965",
                        "baselineColorUp": "#00A04A",
                        "baselineMode": "percentage",
                        "title": {
                            "text": "Revenue",
                            "bold": true,
                            "color": "#434343"
                        },
                        "type": "scorecard",
                        "background": "#FFF7ED",
                        "baseline": "Data!E4",
                        "baselineDescr": "since last period",
                        "keyValue": "Data!D4",
                        "humanize": false
                    }
                },
                {
                    "id": "6510bff8-7b28-42c6-af4b-c750bed2205c",
                    "x": 420,
                    "y": 9,
                    "width": 200,
                    "height": 105,
                    "tag": "chart",
                    "data": {
                        "baselineColorDown": "#DC6965",
                        "baselineColorUp": "#00A04A",
                        "baselineMode": "percentage",
                        "title": {
                            "text": "Attendees",
                            "bold": true,
                            "color": "#434343"
                        },
                        "type": "scorecard",
                        "background": "#EFF6FF",
                        "baseline": "Data!E2",
                        "baselineDescr": "since last period",
                        "keyValue": "Data!D2",
                        "humanize": false
                    }
                },
                {
                    "id": "9757f293-3703-437c-985c-208b91aaa8f1",
                    "x": 0,
                    "y": 188,
                    "width": 472,
                    "height": 335,
                    "tag": "chart",
                    "data": {
                        "type": "bar",
                        "dataSetsHaveTitle": false,
                        "dataSets": [
                            {
                                "dataRange": "Data!B7:B8",
                                "yAxisId": "y"
                            }
                        ],
                        "legendPosition": "none",
                        "labelRange": "Data!A7:A8",
                        "title": {},
                        "stacked": false,
                        "aggregated": false
                    }
                },
                {
                    "id": "9b287910-b65f-4392-889a-52aafce81a3e",
                    "x": 523,
                    "y": 178,
                    "width": 476,
                    "height": 345,
                    "tag": "chart",
                    "data": {
                        "type": "bar",
                        "dataSetsHaveTitle": false,
                        "dataSets": [
                            {
                                "dataRange": "Data!B11:B12",
                                "yAxisId": "y"
                            }
                        ],
                        "legendPosition": "none",
                        "labelRange": "Data!A11:A12",
                        "title": {},
                        "stacked": false,
                        "aggregated": false
                    }
                }
            ],
            "tables": [],
            "areGridLinesVisible": true,
            "isVisible": true,
            "headerGroups": {
                "ROW": [],
                "COL": []
            },
            "dataValidationRules": [],
            "comments": {}
        },
        {
            "id": "3fad37db-0a40-46e9-bcc8-967eebf0bca4",
            "name": "Data",
            "colNumber": 26,
            "rowNumber": 101,
            "rows": {},
            "cols": {},
            "merges": [],
            "cells": {
                "A1": {
                    "content": "=_t(\"KPI\")"
                },
                "A2": {
                    "content": "=_t(\"Attendees\")"
                },
                "A3": {
                    "content": "=_t(\"Events\")"
                },
                "A4": {
                    "content": "=_t(\"Revenue\")"
                },
                "A7": {
                    "content": "=_t(\"Booked\")"
                },
                "A8": {
                    "content": "=_t(\"Ended\")"
                },
                "A11": {
                    "content": "=_t(\"Attended\")"
                },
                "A12": {
                    "content": "=_t(\"Registered\")"
                },
                "B1": {
                    "content": "=_t(\"Current\")"
                },
                "B2": {
                    "content": "478"
                },
                "B3": {
                    "content": "56"
                },
                "B4": {
                    "content": "15497"
                },
                "B7": {
                    "content": "4656"
                },
                "B8": {
                    "content": "435"
                },
                "B11": {
                    "content": "134354"
                },
                "B12": {
                    "content": "32345"
                },
                "C1": {
                    "content": "=_t(\"Previous\")"
                },
                "C2": {
                    "content": "344"
                },
                "C3": {
                    "content": "43"
                },
                "C4": {
                    "content": "11345"
                },
                "D1": {
                    "content": "=_t(\"Current\")"
                },
                "D2": {
                    "content": "=FORMAT.LARGE.NUMBER(B2)"
                },
                "D3": {
                    "content": "=FORMAT.LARGE.NUMBER(B3)"
                },
                "D4": {
                    "content": "=FORMAT.LARGE.NUMBER(B4)"
                },
                "E1": {
                    "content": "=_t(\"Previous\")"
                },
                "E2": {
                    "content": "=FORMAT.LARGE.NUMBER(C2)"
                },
                "E3": {
                    "content": "=FORMAT.LARGE.NUMBER(C3)"
                },
                "E4": {
                    "content": "=FORMAT.LARGE.NUMBER(C4)"
                }
            },
            "styles": {
                "A1:C1": 4,
                "D1:E1": 5,
                "D2:E4": 6
            },
            "formats": {
                "B2:C3": 1,
                "B4:E4": 2
            },
            "borders": {},
            "conditionalFormats": [],
            "figures": [],
            "tables": [],
            "areGridLinesVisible": true,
            "isVisible": true,
            "headerGroups": {
                "ROW": [],
                "COL": []
            },
            "dataValidationRules": [],
            "comments": {}
        }
    ],
    "styles": {
        "1": {
            "textColor": "#01666b",
            "fontSize": 16,
            "bold": true
        },
        "2": {
            "textColor": "#434343",
            "bold": true,
            "fontSize": 11
        },
        "3": {
            "textColor": "#434343",
            "bold": true,
            "fontSize": 11,
            "align": "center"
        },
        "4": {
            "bold": true
        },
        "5": {
            "bold": true,
            "fillColor": "#f2f2f2"
        },
        "6": {
            "fillColor": "#f2f2f2"
        }
    },
    "formats": {
        "1": "0",
        "2": "[$$]#,##0"
    },
    "borders": {
        "1": {
            "bottom": {
                "style": "thin",
                "color": "#CCCCCC"
            }
        },
        "2": {
            "top": {
                "style": "thin",
                "color": "#CCCCCC"
            }
        },
        "3": {
            "bottom": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "right": {
                "style": "thick",
                "color": "#FFFFFF"
            }
        },
        "4": {
            "top": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "bottom": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "right": {
                "style": "thick",
                "color": "#FFFFFF"
            }
        },
        "5": {
            "top": {
                "style": "thick",
                "color": "#FFFFFF"
            }
        },
        "6": {
            "top": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "bottom": {
                "style": "thick",
                "color": "#FFFFFF"
            }
        },
        "7": {
            "bottom": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "left": {
                "style": "thick",
                "color": "#FFFFFF"
            }
        },
        "8": {
            "top": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "bottom": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "left": {
                "style": "thick",
                "color": "#FFFFFF"
            }
        }
    },
    "revisionId": "START_REVISION",
    "uniqueFigureIds": true,
    "settings": {
        "locale": {
            "name": "English (US)",
            "code": "en_US",
            "thousandsSeparator": ",",
            "decimalSeparator": ".",
            "dateFormat": "mm/dd/yyyy",
            "timeFormat": "hh:mm:ss",
            "formulaArgSeparator": ",",
            "weekStart": 7
        }
    },
    "pivots": {},
    "pivotNextId": 11,
    "customTableStyles": {},
    "odooVersion": 12,
    "globalFilters": [],
    "lists": {},
    "listNextId": 1
}

```

