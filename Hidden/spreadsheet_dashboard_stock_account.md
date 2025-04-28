# Odoo Module: spreadsheet_dashboard_stock_account

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
    'name': "Spreadsheet dashboard for stock",
    'category': 'Hidden',
    'summary': 'Spreadsheet',
    'description': 'Spreadsheet',
    'depends': ['spreadsheet_dashboard', 'stock_account'],
    'data': [
        "data/dashboards.xml",
    ],
    'installable': True,
    'auto_install': ['stock_account'],
    'license': 'LGPL-3',
}

```

## File: data\dashboards.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="spreadsheet_dashboard_warehouse_metrics" model="spreadsheet.dashboard">
        <field name="name">Warehouse Metrics</field>
        <field name="spreadsheet_binary_data" type="base64" file="spreadsheet_dashboard_stock_account/data/files/warehouse_metrics_dashboard.json"/>
        <field name="main_data_model_ids" eval="[(4, ref('stock.model_stock_quant'))]"/>
        <field name="sample_dashboard_file_path">spreadsheet_dashboard_stock_account/data/files/warehouse_metrics_sample_dashboard.json</field>
        <field name="dashboard_group_id" ref="spreadsheet_dashboard.spreadsheet_dashboard_group_logistics"/>
        <field name="group_ids" eval="[Command.link(ref('stock.group_stock_manager'))]"/>
        <field name="sequence">300</field>
        <field name="is_published">True</field>
    </record>

</odoo>

```

## File: data\files\warehouse_metrics_dashboard.json

```json
{
    "version": 21,
    "sheets": [
        {
            "id": "sheet1",
            "name": "Dashboard",
            "colNumber": 8,
            "rowNumber": 82,
            "rows": {
                "6": { "size": 38 },
                "22": { "size": 40 },
                "23": { "size": 21 },
                "24": { "size": 21 },
                "25": { "size": 21 },
                "26": { "size": 21 },
                "27": { "size": 21 },
                "28": { "size": 21 },
                "29": { "size": 21 },
                "30": { "size": 21 },
                "31": { "size": 21 },
                "32": { "size": 21 },
                "33": { "size": 21 },
                "34": { "size": 21 },
                "35": { "size": 21 },
                "36": { "size": 21 },
                "37": { "size": 21 },
                "38": { "size": 21 },
                "39": { "size": 21 },
                "40": { "size": 41 },
                "41": { "size": 21 },
                "42": { "size": 21 },
                "43": { "size": 21 },
                "44": { "size": 21 },
                "45": { "size": 21 },
                "46": { "size": 21 },
                "47": { "size": 21 },
                "48": { "size": 21 },
                "49": { "size": 21 },
                "50": { "size": 21 },
                "51": { "size": 21 },
                "52": { "size": 21 },
                "53": { "size": 21 },
                "54": { "size": 21 },
                "55": { "size": 21 },
                "56": { "size": 21 },
                "57": { "size": 36 },
                "58": { "size": 38 },
                "59": { "size": 27 },
                "60": { "size": 27 },
                "61": { "size": 27 },
                "62": { "size": 27 },
                "63": { "size": 27 },
                "64": { "size": 27 },
                "65": { "size": 27 },
                "66": { "size": 27 },
                "67": { "size": 27 },
                "68": { "size": 27 },
                "69": { "size": 21 },
                "70": { "size": 21 },
                "71": { "size": 21 },
                "72": { "size": 21 },
                "73": { "size": 21 },
                "74": { "size": 21 },
                "75": { "size": 21 },
                "76": { "size": 21 },
                "77": { "size": 21 },
                "78": { "size": 21 },
                "79": { "size": 21 },
                "80": { "size": 21 },
                "81": { "size": 21 }
            },
            "cols": {
                "0": { "size": 332 },
                "1": { "size": 100 },
                "2": { "size": 69 },
                "3": { "size": 40 },
                "4": { "size": 50 },
                "5": { "size": 275 },
                "6": { "size": 100 },
                "7": { "size": 55 }
            },
            "merges": [],
            "cells": {
                "A7": {
                    "content": "Available and reserved stock qty (top locations)"
                },
                "A23": {
                    "content": "Available and reserved stock qty (top products)"
                },
                "A41": {
                    "content": "Ageing stock qty by category and creation date"
                },
                "A58": {
                    "content": "=_t(\"Top 10 products with negative stock\")"
                },
                "A59": { "content": "Products" },
                "A60": { "content": "=PIVOT.HEADER(22,\"#product_id\",1)" },
                "A61": { "content": "=PIVOT.HEADER(22,\"#product_id\",2)" },
                "A62": { "content": "=PIVOT.HEADER(22,\"#product_id\",3)" },
                "A63": { "content": "=PIVOT.HEADER(22,\"#product_id\",4)" },
                "A64": { "content": "=PIVOT.HEADER(22,\"#product_id\",5)" },
                "A65": { "content": "=PIVOT.HEADER(22,\"#product_id\",6)" },
                "A66": { "content": "=PIVOT.HEADER(22,\"#product_id\",7)" },
                "A67": { "content": "=PIVOT.HEADER(22,\"#product_id\",8)" },
                "A68": { "content": "=PIVOT.HEADER(22,\"#product_id\",9)" },
                "A69": { "content": "=PIVOT.HEADER(22,\"#product_id\",10)" },
                "B59": {
                    "content": "=PIVOT.HEADER(22,\"measure\",\"quantity\")"
                },
                "B60": {
                    "content": "=PIVOT.VALUE(22,\"quantity\",\"#product_id\",1)"
                },
                "B61": {
                    "content": "=PIVOT.VALUE(22,\"quantity\",\"#product_id\",2)"
                },
                "B62": {
                    "content": "=PIVOT.VALUE(22,\"quantity\",\"#product_id\",3)"
                },
                "B63": {
                    "content": "=PIVOT.VALUE(22,\"quantity\",\"#product_id\",4)"
                },
                "B64": {
                    "content": "=PIVOT.VALUE(22,\"quantity\",\"#product_id\",5)"
                },
                "B65": {
                    "content": "=PIVOT.VALUE(22,\"quantity\",\"#product_id\",6)"
                },
                "B66": {
                    "content": "=PIVOT.VALUE(22,\"quantity\",\"#product_id\",7)"
                },
                "B67": {
                    "content": "=PIVOT.VALUE(22,\"quantity\",\"#product_id\",8)"
                },
                "B68": {
                    "content": "=PIVOT.VALUE(22,\"quantity\",\"#product_id\",9)"
                },
                "B69": {
                    "content": "=PIVOT.VALUE(22,\"quantity\",\"#product_id\",10)"
                },
                "E7": {
                    "content": "Available and reserved stock value (top locations)"
                },
                "E23": {
                    "content": "Available and reserved stock value (top products)"
                },
                "E41": {
                    "content": "Ageing stock value by product and creation date"
                }
            },
            "styles": {
                "A6": 1,
                "A7": 2,
                "A23": 2,
                "A41": 2,
                "A58": 2,
                "D7:E7": 2,
                "E23": 2,
                "E41": 2,
                "A59": 3,
                "A60:B69": 4,
                "B7:C7": 5,
                "B23:D23": 5,
                "B41:D41": 5,
                "F7:H7": 5,
                "F23:H23": 5,
                "F41:H41": 5,
                "B58:H58": 5,
                "B59": 6,
                "C60:H69": 7
            },
            "formats": {},
            "borders": {
                "A7:C7": 1,
                "A23:C23": 1,
                "A41:C41": 1,
                "A58:C58": 1,
                "E7:H7": 1,
                "E23:H23": 1,
                "E41:H41": 1,
                "A8:C8": 2,
                "A24:C24": 2,
                "A42:C42": 2,
                "A59:C59": 2,
                "E8:H8": 2,
                "E24:H24": 2,
                "E42:H42": 2
            },
            "conditionalFormats": [],
            "figures": [
                {
                    "id": "833374d1-d09f-4a4e-bb09-2831ceee5465",
                    "x": 246,
                    "y": 9,
                    "width": 237,
                    "height": 108,
                    "tag": "chart",
                    "data": {
                        "baselineColorDown": "#E06666",
                        "baselineColorUp": "#6AA84F",
                        "baselineMode": "text",
                        "title": {
                            "text": "Share reserved stock Value",
                            "align": "left",
                            "bold": true,
                            "color": "#434343"
                        },
                        "type": "scorecard",
                        "background": "#ECFDF5",
                        "baseline": "Data!E4",
                        "keyValue": "Data!B4",
                        "humanize": false
                    }
                },
                {
                    "id": "de7010e1-2cdc-4e19-a1de-1c7fd4795bf4",
                    "x": 0,
                    "y": 9,
                    "width": 237,
                    "height": 108,
                    "tag": "chart",
                    "data": {
                        "baselineColorDown": "#E06666",
                        "baselineColorUp": "#6AA84F",
                        "baselineMode": "text",
                        "title": {
                            "text": "Share reserved stock Qty",
                            "align": "left",
                            "bold": true,
                            "color": "#434343"
                        },
                        "type": "scorecard",
                        "background": "#ECFDF5",
                        "baseline": "Data!E3",
                        "keyValue": "Data!B3",
                        "humanize": false
                    }
                },
                {
                    "id": "0d527983-ee0f-44cb-af82-f637574a4e1e",
                    "x": 0,
                    "y": 176,
                    "width": 499,
                    "height": 344,
                    "tag": "chart",
                    "data": {
                        "type": "bar",
                        "dataSetsHaveTitle": true,
                        "dataSets": [
                            { "dataRange": "'Stock location Qty'!B1:B11" },
                            { "dataRange": "'Stock location Qty'!C1:C11" }
                        ],
                        "legendPosition": "top",
                        "labelRange": "'Stock location Qty'!A1:A11",
                        "title": { "text": "" },
                        "stacked": true,
                        "aggregated": false
                    }
                },
                {
                    "id": "e7f90aaf-95ec-4f1d-9b0c-ec7e4465d772",
                    "x": 0,
                    "y": 561,
                    "width": 501,
                    "height": 355,
                    "tag": "chart",
                    "data": {
                        "type": "bar",
                        "dataSetsHaveTitle": true,
                        "dataSets": [
                            { "dataRange": "'Stock location Qty'!F1:F11" },
                            { "dataRange": "'Stock location Qty'!G1:G11" }
                        ],
                        "legendPosition": "top",
                        "labelRange": "'Stock location Qty'!E1:E11",
                        "title": { "text": "" },
                        "stacked": true,
                        "aggregated": false
                    }
                },
                {
                    "id": "8324b219-c649-4392-9fee-a3e7f396d70c",
                    "x": 540,
                    "y": 176,
                    "width": 483,
                    "height": 344,
                    "tag": "chart",
                    "data": {
                        "type": "bar",
                        "dataSetsHaveTitle": true,
                        "dataSets": [
                            { "dataRange": "'Stock location Value'!B1:B12" },
                            { "dataRange": "'Stock location Value'!C1:C12" }
                        ],
                        "legendPosition": "top",
                        "labelRange": "'Stock location Value'!A1:A12",
                        "title": { "text": "" },
                        "stacked": true,
                        "aggregated": false
                    }
                },
                {
                    "id": "ebde3556-983a-475a-95f0-4edb2515b3bb",
                    "x": 540,
                    "y": 561,
                    "width": 482,
                    "height": 358,
                    "tag": "chart",
                    "data": {
                        "type": "bar",
                        "dataSetsHaveTitle": true,
                        "dataSets": [
                            { "dataRange": "'Stock location Value'!I1:I11" },
                            { "dataRange": "'Stock location Value'!J1:J11" }
                        ],
                        "legendPosition": "top",
                        "labelRange": "'Stock location Value'!H1:H11",
                        "title": { "text": "" },
                        "stacked": true,
                        "aggregated": false
                    }
                },
                {
                    "id": "fad86b76-037f-4b7f-8afc-bc028c2536de",
                    "x": 0,
                    "y": 959,
                    "width": 500,
                    "height": 335,
                    "tag": "chart",
                    "data": {
                        "title": { "text": "" },
                        "background": "#FFFFFF",
                        "legendPosition": "none",
                        "metaData": {
                            "groupBy": ["create_date:quarter", "product_id"],
                            "measure": "quantity",
                            "order": null,
                            "resModel": "stock.quant",
                            "mode": "bar"
                        },
                        "searchParams": {
                            "comparison": null,
                            "context": {
                                "mail_notify_force_send": false,
                                "always_show_loc": 1,
                                "inventory_mode": true,
                                "inventory_report_mode": true
                            },
                            "domain": [["location_id.usage", "=", "internal"]],
                            "groupBy": ["create_date:quarter", "product_id"],
                            "orderBy": []
                        },
                        "type": "odoo_bar",
                        "verticalAxisPosition": "left",
                        "stacked": true,
                        "fieldMatching": {
                            "feeb0d08-4596-4e7c-b478-a3c659189cdd": {
                                "chain": "warehouse_id",
                                "type": "many2one"
                            },
                            "62f6d1bd-9c76-48d7-b95a-3f5d6cb505df": {
                                "chain": "location_id",
                                "type": "many2one"
                            },
                            "7ff87378-db1d-4725-bf25-6eee704b15a5": {
                                "chain": "product_categ_id",
                                "type": "many2one"
                            },
                            "8dea5478-ff09-41fc-ada0-cf23c66e4f8b": {
                                "chain": "product_id",
                                "type": "many2one"
                            },
                            "0725cf6d-c980-46a1-bd7b-bd75e34348e7": {
                                "chain": "lot_id",
                                "type": "many2one"
                            }
                        }
                    }
                },
                {
                    "id": "73f61914-d895-4484-b3ba-585107daeb44",
                    "x": 541,
                    "y": 959,
                    "width": 480,
                    "height": 335,
                    "tag": "chart",
                    "data": {
                        "title": { "text": "" },
                        "background": "#FFFFFF",
                        "legendPosition": "none",
                        "metaData": {
                            "groupBy": ["create_date:quarter", "product_id"],
                            "measure": "value",
                            "order": null,
                            "resModel": "stock.quant",
                            "mode": "bar"
                        },
                        "searchParams": {
                            "comparison": null,
                            "context": {
                                "mail_notify_force_send": false,
                                "always_show_loc": 1,
                                "inventory_mode": true,
                                "inventory_report_mode": true
                            },
                            "domain": [["location_id.usage", "=", "internal"]],
                            "groupBy": ["create_date:quarter", "product_id"],
                            "orderBy": []
                        },
                        "type": "odoo_bar",
                        "verticalAxisPosition": "left",
                        "stacked": true,
                        "fieldMatching": {
                            "feeb0d08-4596-4e7c-b478-a3c659189cdd": {
                                "chain": "warehouse_id",
                                "type": "many2one"
                            },
                            "62f6d1bd-9c76-48d7-b95a-3f5d6cb505df": {
                                "chain": "location_id",
                                "type": "many2one"
                            },
                            "7ff87378-db1d-4725-bf25-6eee704b15a5": {
                                "chain": "product_categ_id",
                                "type": "many2one"
                            },
                            "8dea5478-ff09-41fc-ada0-cf23c66e4f8b": {
                                "chain": "product_id",
                                "type": "many2one"
                            },
                            "0725cf6d-c980-46a1-bd7b-bd75e34348e7": {
                                "chain": "lot_id",
                                "type": "many2one"
                            }
                        }
                    }
                },
                {
                    "id": "639d221d-d74d-4762-984e-9debac4f5a82",
                    "x": 492,
                    "y": 9,
                    "width": 237,
                    "height": 108,
                    "tag": "chart",
                    "data": {
                        "baselineColorDown": "#E06666",
                        "baselineColorUp": "#6AA84F",
                        "baselineMode": "difference",
                        "title": {
                            "text": "Lines with negative stock",
                            "bold": true,
                            "align": "left",
                            "color": "#434343"
                        },
                        "type": "scorecard",
                        "background": "#EFF6FF",
                        "keyValue": "Data!B5",
                        "humanize": true
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
            "id": "fb6d5d91-04cf-4c22-953a-a00c4e8f19e4",
            "name": "Data",
            "colNumber": 22,
            "rowNumber": 82,
            "rows": {},
            "cols": {
                "0": { "size": 227 },
                "1": { "size": 136 },
                "2": { "size": 111 },
                "3": { "size": 113 },
                "4": { "size": 205 },
                "6": { "size": 107 }
            },
            "merges": [],
            "cells": {
                "A1": { "content": "=_t(\"KPI\")" },
                "A2": { "content": "=_t(\"Total inventory value\")" },
                "A3": { "content": "Share of reserved stock qty" },
                "A4": { "content": "Share of reserved stock Value" },
                "A5": { "content": "Count of products with negative stock" },
                "B2": {
                    "content": "=FORMAT.LARGE.NUMBER(PIVOT.VALUE(20,\"value\"))"
                },
                "B3": {
                    "content": "=iferror(PIVOT.VALUE(20,\"reserved_quantity\")/PIVOT.VALUE(20,\"quantity\"),\"No data\")"
                },
                "B4": { "content": "=IFERROR(C4/D4,\"No data\")" },
                "B5": { "content": "=PIVOT.VALUE(22,\"__count\")" },
                "C1": { "content": "Reserved" },
                "C3": { "content": "=PIVOT.VALUE(20,\"reserved_quantity\")" },
                "C4": {
                    "content": "=IFERROR(FORMAT.LARGE.NUMBER(round(PIVOT.VALUE(18,\"value\")/PIVOT.VALUE(18,\"quantity\"))*PIVOT.VALUE(18,\"reserved_quantity\")),\"No data\")"
                },
                "D1": { "content": "Total" },
                "D3": { "content": "=PIVOT.VALUE(20,\"quantity\")" },
                "D4": {
                    "content": "=FORMAT.LARGE.NUMBER(PIVOT.VALUE(20,\"value\"))"
                },
                "E3": {
                    "content": "=TEXT(C3,\"#,##0\")&\" out of \" &TEXT(D3,\"#,##0\")"
                },
                "E4": {
                    "content": "=Iferror(TEXT(C4,\"#,##0\")&\" out of \" &TEXT(D4,\"#,##0\"),\"No data\")"
                }
            },
            "styles": { "A1:D1": 8, "A2:A5": 9 },
            "formats": { "B3:B4": 1, "B5:C5": 2 },
            "borders": {},
            "conditionalFormats": [],
            "figures": [],
            "tables": [],
            "areGridLinesVisible": true,
            "isVisible": true,
            "headerGroups": { "ROW": [], "COL": [] },
            "dataValidationRules": [],
            "comments": {}
        },
        {
            "id": "ff7cd299-81f4-49db-a27a-66bf36420f0d",
            "name": "Stock location Qty",
            "colNumber": 12,
            "rowNumber": 99,
            "rows": {},
            "cols": {
                "0": { "size": 134 },
                "1": { "size": 112 },
                "2": { "size": 115 },
                "3": { "size": 55 },
                "4": { "size": 316 },
                "5": { "size": 112 },
                "6": { "size": 115 }
            },
            "merges": [],
            "cells": {
                "A1": { "content": "Location" },
                "A2": { "content": "=PIVOT.HEADER(8,\"#location_id\",1)" },
                "A3": { "content": "=PIVOT.HEADER(8,\"#location_id\",2)" },
                "A4": { "content": "=PIVOT.HEADER(8,\"#location_id\",3)" },
                "A5": { "content": "=PIVOT.HEADER(8,\"#location_id\",4)" },
                "A6": { "content": "=PIVOT.HEADER(8,\"#location_id\",5)" },
                "A7": { "content": "=PIVOT.HEADER(8,\"#location_id\",6)" },
                "A8": { "content": "=PIVOT.HEADER(8,\"#location_id\",7)" },
                "A9": { "content": "=PIVOT.HEADER(8,\"#location_id\",8)" },
                "A10": { "content": "=PIVOT.HEADER(8,\"#location_id\",9)" },
                "A11": { "content": "=PIVOT.HEADER(8,\"#location_id\",10)" },
                "B1": {
                    "content": "=PIVOT.HEADER(9,\"measure\",\"available_quantity\")"
                },
                "B2": {
                    "content": "=PIVOT.VALUE(8,\"available_quantity\",\"#location_id\",1)"
                },
                "B3": {
                    "content": "=PIVOT.VALUE(8,\"available_quantity\",\"#location_id\",2)"
                },
                "B4": {
                    "content": "=PIVOT.VALUE(8,\"available_quantity\",\"#location_id\",3)"
                },
                "B5": {
                    "content": "=PIVOT.VALUE(8,\"available_quantity\",\"#location_id\",4)"
                },
                "B6": {
                    "content": "=PIVOT.VALUE(8,\"available_quantity\",\"#location_id\",5)"
                },
                "B7": {
                    "content": "=PIVOT.VALUE(8,\"available_quantity\",\"#location_id\",6)"
                },
                "B8": {
                    "content": "=PIVOT.VALUE(8,\"available_quantity\",\"#location_id\",7)"
                },
                "B9": {
                    "content": "=PIVOT.VALUE(8,\"available_quantity\",\"#location_id\",8)"
                },
                "B10": {
                    "content": "=PIVOT.VALUE(8,\"available_quantity\",\"#location_id\",9)"
                },
                "B11": {
                    "content": "=PIVOT.VALUE(8,\"available_quantity\",\"#location_id\",10)"
                },
                "C1": {
                    "content": "=PIVOT.HEADER(9,\"measure\",\"reserved_quantity\")"
                },
                "C2": {
                    "content": "=PIVOT.VALUE(8,\"reserved_quantity\",\"#location_id\",1)"
                },
                "C3": {
                    "content": "=PIVOT.VALUE(8,\"reserved_quantity\",\"#location_id\",2)"
                },
                "C4": {
                    "content": "=PIVOT.VALUE(8,\"reserved_quantity\",\"#location_id\",3)"
                },
                "C5": {
                    "content": "=PIVOT.VALUE(8,\"reserved_quantity\",\"#location_id\",4)"
                },
                "C6": {
                    "content": "=PIVOT.VALUE(8,\"reserved_quantity\",\"#location_id\",5)"
                },
                "C7": {
                    "content": "=PIVOT.VALUE(8,\"reserved_quantity\",\"#location_id\",6)"
                },
                "C8": {
                    "content": "=PIVOT.VALUE(8,\"reserved_quantity\",\"#location_id\",7)"
                },
                "C9": {
                    "content": "=PIVOT.VALUE(8,\"reserved_quantity\",\"#location_id\",8)"
                },
                "C10": {
                    "content": "=PIVOT.VALUE(8,\"reserved_quantity\",\"#location_id\",9)"
                },
                "C11": {
                    "content": "=PIVOT.VALUE(8,\"reserved_quantity\",\"#location_id\",10)"
                },
                "E2": { "content": "=PIVOT.HEADER(9,\"#product_id\",1)" },
                "E3": { "content": "=PIVOT.HEADER(9,\"#product_id\",2)" },
                "E4": { "content": "=PIVOT.HEADER(9,\"#product_id\",3)" },
                "E5": { "content": "=PIVOT.HEADER(9,\"#product_id\",4)" },
                "E6": { "content": "=PIVOT.HEADER(9,\"#product_id\",5)" },
                "E7": { "content": "=PIVOT.HEADER(9,\"#product_id\",6)" },
                "E8": { "content": "=PIVOT.HEADER(9,\"#product_id\",7)" },
                "E9": { "content": "=PIVOT.HEADER(9,\"#product_id\",8)" },
                "E10": { "content": "=PIVOT.HEADER(9,\"#product_id\",9)" },
                "E11": { "content": "=PIVOT.HEADER(9,\"#product_id\",10)" },
                "F1": {
                    "content": "=PIVOT.HEADER(9,\"measure\",\"available_quantity\")"
                },
                "F2": {
                    "content": "=PIVOT.VALUE(9,\"quantity\",\"#product_id\",1)"
                },
                "F3": {
                    "content": "=PIVOT.VALUE(9,\"quantity\",\"#product_id\",2)"
                },
                "F4": {
                    "content": "=PIVOT.VALUE(9,\"quantity\",\"#product_id\",3)"
                },
                "F5": {
                    "content": "=PIVOT.VALUE(9,\"quantity\",\"#product_id\",4)"
                },
                "F6": {
                    "content": "=PIVOT.VALUE(9,\"quantity\",\"#product_id\",5)"
                },
                "F7": {
                    "content": "=PIVOT.VALUE(9,\"quantity\",\"#product_id\",6)"
                },
                "F8": {
                    "content": "=PIVOT.VALUE(9,\"quantity\",\"#product_id\",7)"
                },
                "F9": {
                    "content": "=PIVOT.VALUE(9,\"quantity\",\"#product_id\",8)"
                },
                "F10": {
                    "content": "=PIVOT.VALUE(9,\"quantity\",\"#product_id\",9)"
                },
                "F11": {
                    "content": "=PIVOT.VALUE(9,\"quantity\",\"#product_id\",10)"
                },
                "G1": {
                    "content": "=PIVOT.HEADER(9,\"measure\",\"reserved_quantity\")"
                },
                "G2": {
                    "content": "=PIVOT.VALUE(9,\"reserved_quantity\",\"#product_id\",1)"
                },
                "G3": {
                    "content": "=PIVOT.VALUE(9,\"reserved_quantity\",\"#product_id\",2)"
                },
                "G4": {
                    "content": "=PIVOT.VALUE(9,\"reserved_quantity\",\"#product_id\",3)"
                },
                "G5": {
                    "content": "=PIVOT.VALUE(9,\"reserved_quantity\",\"#product_id\",4)"
                },
                "G6": {
                    "content": "=PIVOT.VALUE(9,\"reserved_quantity\",\"#product_id\",5)"
                },
                "G7": {
                    "content": "=PIVOT.VALUE(9,\"reserved_quantity\",\"#product_id\",6)"
                },
                "G8": {
                    "content": "=PIVOT.VALUE(9,\"reserved_quantity\",\"#product_id\",7)"
                },
                "G9": {
                    "content": "=PIVOT.VALUE(9,\"reserved_quantity\",\"#product_id\",8)"
                },
                "G10": {
                    "content": "=PIVOT.VALUE(9,\"reserved_quantity\",\"#product_id\",9)"
                },
                "G11": {
                    "content": "=PIVOT.VALUE(9,\"reserved_quantity\",\"#product_id\",10)"
                }
            },
            "styles": {},
            "formats": { "F2:G12": 2 },
            "borders": {},
            "conditionalFormats": [],
            "figures": [],
            "tables": [
                {
                    "range": "A1:C11",
                    "type": "static",
                    "config": {
                        "hasFilters": false,
                        "totalRow": false,
                        "firstColumn": true,
                        "lastColumn": false,
                        "numberOfHeaders": 1,
                        "bandedRows": true,
                        "bandedColumns": false,
                        "styleId": "TableStyleMedium5"
                    }
                },
                {
                    "range": "E1:G11",
                    "type": "static",
                    "config": {
                        "hasFilters": false,
                        "totalRow": false,
                        "firstColumn": true,
                        "lastColumn": false,
                        "numberOfHeaders": 1,
                        "bandedRows": true,
                        "bandedColumns": false,
                        "styleId": "TableStyleMedium5"
                    }
                }
            ],
            "areGridLinesVisible": true,
            "isVisible": true,
            "headerGroups": { "ROW": [], "COL": [] },
            "dataValidationRules": [],
            "comments": {}
        },
        {
            "id": "b87c927f-558d-4244-b0e0-76f0cc2e3467",
            "name": "Stock location Value",
            "colNumber": 28,
            "rowNumber": 99,
            "rows": {},
            "cols": {
                "0": { "size": 113 },
                "1": { "size": 98 },
                "2": { "size": 97 },
                "3": { "size": 72 },
                "4": { "size": 72 },
                "5": { "size": 115 },
                "6": { "size": 92 },
                "7": { "size": 207 },
                "8": { "size": 139 },
                "9": { "size": 139 },
                "10": { "size": 139 },
                "11": { "size": 139 },
                "12": { "size": 139 },
                "13": { "size": 139 },
                "14": { "size": 139 },
                "15": { "size": 139 },
                "16": { "size": 139 },
                "17": { "size": 139 },
                "18": { "size": 139 },
                "19": { "size": 139 },
                "20": { "size": 139 },
                "21": { "size": 139 },
                "22": { "size": 139 },
                "23": { "size": 139 },
                "24": { "size": 139 },
                "25": { "size": 139 },
                "26": { "size": 139 },
                "27": { "size": 139 }
            },
            "merges": [],
            "cells": {
                "A2": {
                    "content": "=PIVOT.HEADER(21,\"#location_id\",row()-1)"
                },
                "A3": {
                    "content": "=PIVOT.HEADER(21,\"#location_id\",row()-1)"
                },
                "A4": {
                    "content": "=PIVOT.HEADER(21,\"#location_id\",row()-1)"
                },
                "A5": {
                    "content": "=PIVOT.HEADER(21,\"#location_id\",row()-1)"
                },
                "A6": {
                    "content": "=PIVOT.HEADER(21,\"#location_id\",row()-1)"
                },
                "A7": {
                    "content": "=PIVOT.HEADER(21,\"#location_id\",row()-1)"
                },
                "A8": {
                    "content": "=PIVOT.HEADER(21,\"#location_id\",row()-1)"
                },
                "A9": {
                    "content": "=PIVOT.HEADER(21,\"#location_id\",row()-1)"
                },
                "A10": {
                    "content": "=PIVOT.HEADER(21,\"#location_id\",row()-1)"
                },
                "A11": {
                    "content": "=PIVOT.HEADER(21,\"#location_id\",row()-1)"
                },
                "A12": {
                    "content": "=PIVOT.HEADER(21,\"#location_id\",row()-1)"
                },
                "B1": { "content": "Available Value" },
                "B2": { "content": "=if(D2<>\"\",D2-C2,\"\")" },
                "B3": { "content": "=if(D3<>\"\",D3-C3,\"\")" },
                "B4": { "content": "=if(D4<>\"\",D4-C4,\"\")" },
                "B5": { "content": "=if(D5<>\"\",D5-C5,\"\")" },
                "B6": { "content": "=if(D6<>\"\",D6-C6,\"\")" },
                "B7": { "content": "=if(D7<>\"\",D7-C7,\"\")" },
                "B8": { "content": "=if(D8<>\"\",D8-C8,\"\")" },
                "B9": { "content": "=if(D9<>\"\",D9-C9,\"\")" },
                "B10": { "content": "=if(D10<>\"\",D10-C10,\"\")" },
                "B11": { "content": "=if(D11<>\"\",D11-C11,\"\")" },
                "B12": { "content": "=if(D12<>\"\",D12-C12,\"\")" },
                "C1": { "content": "Reserved value" },
                "C2": {
                    "content": "=if( AND(F2>0,A2<>\"\"), IFERROR(FORMAT.LARGE.NUMBER(round(PIVOT.VALUE(23,\"value\",\"#location_id\",row()-1)/PIVOT.VALUE(23,\"quantity\",\"#location_id\",row()-1))*PIVOT.VALUE(23,\"reserved_quantity\",\"#location_id\",row()-1)),\"No data\"),\"\")"
                },
                "C3": {
                    "content": "=if( AND(F3>0,A3<>\"\"), IFERROR(FORMAT.LARGE.NUMBER(round(PIVOT.VALUE(23,\"value\",\"#location_id\",row()-1)/PIVOT.VALUE(23,\"quantity\",\"#location_id\",row()-1))*PIVOT.VALUE(23,\"reserved_quantity\",\"#location_id\",row()-1)),\"No data\"),\"\")"
                },
                "C4": {
                    "content": "=if( AND(F4>0,A4<>\"\"), IFERROR(FORMAT.LARGE.NUMBER(round(PIVOT.VALUE(23,\"value\",\"#location_id\",row()-1)/PIVOT.VALUE(23,\"quantity\",\"#location_id\",row()-1))*PIVOT.VALUE(23,\"reserved_quantity\",\"#location_id\",row()-1)),\"No data\"),\"\")"
                },
                "C5": {
                    "content": "=if( AND(F5>0,A5<>\"\"), IFERROR(FORMAT.LARGE.NUMBER(round(PIVOT.VALUE(23,\"value\",\"#location_id\",row()-1)/PIVOT.VALUE(23,\"quantity\",\"#location_id\",row()-1))*PIVOT.VALUE(23,\"reserved_quantity\",\"#location_id\",row()-1)),\"No data\"),\"\")"
                },
                "C6": {
                    "content": "=if( AND(F6>0,A6<>\"\"), IFERROR(FORMAT.LARGE.NUMBER(round(PIVOT.VALUE(23,\"value\",\"#location_id\",row()-1)/PIVOT.VALUE(23,\"quantity\",\"#location_id\",row()-1))*PIVOT.VALUE(23,\"reserved_quantity\",\"#location_id\",row()-1)),\"No data\"),\"\")"
                },
                "C7": {
                    "content": "=if( AND(F7>0,A7<>\"\"), IFERROR(FORMAT.LARGE.NUMBER(round(PIVOT.VALUE(23,\"value\",\"#location_id\",row()-1)/PIVOT.VALUE(23,\"quantity\",\"#location_id\",row()-1))*PIVOT.VALUE(23,\"reserved_quantity\",\"#location_id\",row()-1)),\"No data\"),\"\")"
                },
                "C8": {
                    "content": "=if( AND(F8>0,A8<>\"\"), IFERROR(FORMAT.LARGE.NUMBER(round(PIVOT.VALUE(23,\"value\",\"#location_id\",row()-1)/PIVOT.VALUE(23,\"quantity\",\"#location_id\",row()-1))*PIVOT.VALUE(23,\"reserved_quantity\",\"#location_id\",row()-1)),\"No data\"),\"\")"
                },
                "C9": {
                    "content": "=if( AND(F9>0,A9<>\"\"), IFERROR(FORMAT.LARGE.NUMBER(round(PIVOT.VALUE(23,\"value\",\"#location_id\",row()-1)/PIVOT.VALUE(23,\"quantity\",\"#location_id\",row()-1))*PIVOT.VALUE(23,\"reserved_quantity\",\"#location_id\",row()-1)),\"No data\"),\"\")"
                },
                "C10": {
                    "content": "=if( AND(F10>0,A10<>\"\"), IFERROR(FORMAT.LARGE.NUMBER(round(PIVOT.VALUE(23,\"value\",\"#location_id\",row()-1)/PIVOT.VALUE(23,\"quantity\",\"#location_id\",row()-1))*PIVOT.VALUE(23,\"reserved_quantity\",\"#location_id\",row()-1)),\"No data\"),\"\")"
                },
                "C11": {
                    "content": "=if( AND(F11>0,A11<>\"\"), IFERROR(FORMAT.LARGE.NUMBER(round(PIVOT.VALUE(23,\"value\",\"#location_id\",row()-1)/PIVOT.VALUE(23,\"quantity\",\"#location_id\",row()-1))*PIVOT.VALUE(23,\"reserved_quantity\",\"#location_id\",row()-1)),\"No data\"),\"\")"
                },
                "C12": {
                    "content": "=if( AND(F12>0,A12<>\"\"), IFERROR(FORMAT.LARGE.NUMBER(round(PIVOT.VALUE(23,\"value\",\"#location_id\",row()-1)/PIVOT.VALUE(23,\"quantity\",\"#location_id\",row()-1))*PIVOT.VALUE(23,\"reserved_quantity\",\"#location_id\",row()-1)),\"No data\"),\"\")"
                },
                "D1": { "content": "=PIVOT.HEADER(21,\"measure\",\"value\")" },
                "D2": {
                    "content": "=PIVOT.VALUE(21,\"value\",\"#location_id\",1)"
                },
                "D3": {
                    "content": "=PIVOT.VALUE(21,\"value\",\"#location_id\",2)"
                },
                "D4": {
                    "content": "=PIVOT.VALUE(21,\"value\",\"#location_id\",3)"
                },
                "D5": {
                    "content": "=PIVOT.VALUE(21,\"value\",\"#location_id\",4)"
                },
                "D6": {
                    "content": "=PIVOT.VALUE(21,\"value\",\"#location_id\",5)"
                },
                "D7": {
                    "content": "=PIVOT.VALUE(21,\"value\",\"#location_id\",6)"
                },
                "D8": {
                    "content": "=PIVOT.VALUE(21,\"value\",\"#location_id\",7)"
                },
                "D9": {
                    "content": "=PIVOT.VALUE(21,\"value\",\"#location_id\",8)"
                },
                "D10": {
                    "content": "=PIVOT.VALUE(21,\"value\",\"#location_id\",9)"
                },
                "D11": {
                    "content": "=PIVOT.VALUE(21,\"value\",\"#location_id\",10)"
                },
                "D12": {
                    "content": "=PIVOT.VALUE(21,\"value\",\"#location_id\",11)"
                },
                "E1": {
                    "content": "=PIVOT.HEADER(21,\"measure\",\"quantity\")"
                },
                "E2": {
                    "content": "=PIVOT.VALUE(21,\"quantity\",\"#location_id\",1)"
                },
                "E3": {
                    "content": "=PIVOT.VALUE(21,\"quantity\",\"#location_id\",2)"
                },
                "E4": {
                    "content": "=PIVOT.VALUE(21,\"quantity\",\"#location_id\",3)"
                },
                "E5": {
                    "content": "=PIVOT.VALUE(21,\"quantity\",\"#location_id\",4)"
                },
                "E6": {
                    "content": "=PIVOT.VALUE(21,\"quantity\",\"#location_id\",5)"
                },
                "E7": {
                    "content": "=PIVOT.VALUE(21,\"quantity\",\"#location_id\",6)"
                },
                "E8": {
                    "content": "=PIVOT.VALUE(21,\"quantity\",\"#location_id\",7)"
                },
                "E9": {
                    "content": "=PIVOT.VALUE(21,\"quantity\",\"#location_id\",8)"
                },
                "E10": {
                    "content": "=PIVOT.VALUE(21,\"quantity\",\"#location_id\",9)"
                },
                "E11": {
                    "content": "=PIVOT.VALUE(21,\"quantity\",\"#location_id\",10)"
                },
                "E12": {
                    "content": "=PIVOT.VALUE(21,\"quantity\",\"#location_id\",11)"
                },
                "F1": {
                    "content": "=PIVOT.HEADER(21,\"measure\",\"reserved_quantity\")"
                },
                "F2": {
                    "content": "=PIVOT.VALUE(8,\"reserved_quantity\",\"#location_id\",1)"
                },
                "F3": {
                    "content": "=PIVOT.VALUE(21,\"reserved_quantity\",\"#location_id\",2)"
                },
                "F4": {
                    "content": "=PIVOT.VALUE(21,\"reserved_quantity\",\"#location_id\",3)"
                },
                "F5": {
                    "content": "=PIVOT.VALUE(21,\"reserved_quantity\",\"#location_id\",4)"
                },
                "F6": {
                    "content": "=PIVOT.VALUE(21,\"reserved_quantity\",\"#location_id\",5)"
                },
                "F7": {
                    "content": "=PIVOT.VALUE(21,\"reserved_quantity\",\"#location_id\",6)"
                },
                "F8": {
                    "content": "=PIVOT.VALUE(21,\"reserved_quantity\",\"#location_id\",7)"
                },
                "F9": {
                    "content": "=PIVOT.VALUE(21,\"reserved_quantity\",\"#location_id\",8)"
                },
                "F10": {
                    "content": "=PIVOT.VALUE(21,\"reserved_quantity\",\"#location_id\",9)"
                },
                "F11": {
                    "content": "=PIVOT.VALUE(21,\"reserved_quantity\",\"#location_id\",10)"
                },
                "F12": {
                    "content": "=PIVOT.VALUE(21,\"reserved_quantity\",\"#location_id\",11)"
                },
                "H2": { "content": "=PIVOT.HEADER(20,\"#product_id\",1)" },
                "H3": { "content": "=PIVOT.HEADER(20,\"#product_id\",2)" },
                "H4": { "content": "=PIVOT.HEADER(20,\"#product_id\",3)" },
                "H5": { "content": "=PIVOT.HEADER(20,\"#product_id\",4)" },
                "H6": { "content": "=PIVOT.HEADER(20,\"#product_id\",5)" },
                "H7": { "content": "=PIVOT.HEADER(20,\"#product_id\",6)" },
                "H8": { "content": "=PIVOT.HEADER(20,\"#product_id\",7)" },
                "H9": { "content": "=PIVOT.HEADER(20,\"#product_id\",8)" },
                "H10": { "content": "=PIVOT.HEADER(20,\"#product_id\",9)" },
                "H11": { "content": "=PIVOT.HEADER(20,\"#product_id\",10)" },
                "I1": { "content": "Available Value" },
                "I2": { "content": "=if(K2<>\"\",K2-J2,\"\")" },
                "I3": { "content": "=if(K3<>\"\",K3-J3,\"\")" },
                "I4": { "content": "=if(K4<>\"\",K4-J4,\"\")" },
                "I5": { "content": "=if(K5<>\"\",K5-J5,\"\")" },
                "I6": { "content": "=if(K6<>\"\",K6-J6,\"\")" },
                "I7": { "content": "=if(K7<>\"\",K7-J7,\"\")" },
                "I8": { "content": "=if(K8<>\"\",K8-J8,\"\")" },
                "I9": { "content": "=if(K9<>\"\",K9-J9,\"\")" },
                "I10": { "content": "=if(K10<>\"\",K10-J10,\"\")" },
                "I11": { "content": "=if(K11<>\"\",K11-J11,\"\")" },
                "J1": { "content": "Reserved value" },
                "J2": { "content": "=(M2/L2)*K2" },
                "J3": { "content": "=(M3/L3)*K3" },
                "J4": { "content": "=(M4/L4)*K4" },
                "J5": { "content": "=(M5/L5)*K5" },
                "J6": { "content": "=(M6/L6)*K6" },
                "J7": { "content": "=(M7/L7)*K7" },
                "J8": { "content": "=(M8/L8)*K8" },
                "J9": { "content": "=(M9/L9)*K9" },
                "J10": { "content": "=(M10/L10)*K10" },
                "J11": { "content": "=(M11/L11)*K11" },
                "K1": { "content": "=PIVOT.HEADER(20,\"measure\",\"value\")" },
                "K2": {
                    "content": "=PIVOT.VALUE(20,\"value\",\"#product_id\",row()-1)"
                },
                "K3": {
                    "content": "=PIVOT.VALUE(20,\"value\",\"#product_id\",row()-1)"
                },
                "K4": {
                    "content": "=PIVOT.VALUE(20,\"value\",\"#product_id\",row()-1)"
                },
                "K5": {
                    "content": "=PIVOT.VALUE(20,\"value\",\"#product_id\",row()-1)"
                },
                "K6": {
                    "content": "=PIVOT.VALUE(20,\"value\",\"#product_id\",row()-1)"
                },
                "K7": {
                    "content": "=PIVOT.VALUE(20,\"value\",\"#product_id\",row()-1)"
                },
                "K8": {
                    "content": "=PIVOT.VALUE(20,\"value\",\"#product_id\",row()-1)"
                },
                "K9": {
                    "content": "=PIVOT.VALUE(20,\"value\",\"#product_id\",row()-1)"
                },
                "K10": {
                    "content": "=PIVOT.VALUE(20,\"value\",\"#product_id\",row()-1)"
                },
                "K11": {
                    "content": "=PIVOT.VALUE(20,\"value\",\"#product_id\",row()-1)"
                },
                "L1": {
                    "content": "=PIVOT.HEADER(20,\"measure\",\"quantity\")"
                },
                "L2": {
                    "content": "=PIVOT.VALUE(20,\"quantity\",\"#product_id\",1)"
                },
                "L3": {
                    "content": "=PIVOT.VALUE(20,\"quantity\",\"#product_id\",2)"
                },
                "L4": {
                    "content": "=PIVOT.VALUE(20,\"quantity\",\"#product_id\",3)"
                },
                "L5": {
                    "content": "=PIVOT.VALUE(20,\"quantity\",\"#product_id\",4)"
                },
                "L6": {
                    "content": "=PIVOT.VALUE(20,\"quantity\",\"#product_id\",5)"
                },
                "L7": {
                    "content": "=PIVOT.VALUE(20,\"quantity\",\"#product_id\",6)"
                },
                "L8": {
                    "content": "=PIVOT.VALUE(20,\"quantity\",\"#product_id\",7)"
                },
                "L9": {
                    "content": "=PIVOT.VALUE(20,\"quantity\",\"#product_id\",8)"
                },
                "L10": {
                    "content": "=PIVOT.VALUE(20,\"quantity\",\"#product_id\",9)"
                },
                "L11": {
                    "content": "=PIVOT.VALUE(20,\"quantity\",\"#product_id\",10)"
                },
                "M1": {
                    "content": "=PIVOT.HEADER(20,\"measure\",\"reserved_quantity\")"
                },
                "M2": {
                    "content": "=PIVOT.VALUE(20,\"reserved_quantity\",\"#product_id\",1)"
                },
                "M3": {
                    "content": "=PIVOT.VALUE(20,\"reserved_quantity\",\"#product_id\",2)"
                },
                "M4": {
                    "content": "=PIVOT.VALUE(20,\"reserved_quantity\",\"#product_id\",3)"
                },
                "M5": {
                    "content": "=PIVOT.VALUE(20,\"reserved_quantity\",\"#product_id\",4)"
                },
                "M6": {
                    "content": "=PIVOT.VALUE(20,\"reserved_quantity\",\"#product_id\",5)"
                },
                "M7": {
                    "content": "=PIVOT.VALUE(20,\"reserved_quantity\",\"#product_id\",6)"
                },
                "M8": {
                    "content": "=PIVOT.VALUE(20,\"reserved_quantity\",\"#product_id\",7)"
                },
                "M9": {
                    "content": "=PIVOT.VALUE(20,\"reserved_quantity\",\"#product_id\",8)"
                },
                "M10": {
                    "content": "=PIVOT.VALUE(20,\"reserved_quantity\",\"#product_id\",9)"
                },
                "M11": {
                    "content": "=PIVOT.VALUE(20,\"reserved_quantity\",\"#product_id\",10)"
                }
            },
            "styles": {},
            "formats": {
                "D4:E11": 2,
                "B2:B3": 3,
                "C2:C12": 3,
                "D2:F3": 3,
                "J2:J11": 3
            },
            "borders": {},
            "conditionalFormats": [],
            "figures": [],
            "tables": [
                {
                    "range": "H1:M11",
                    "type": "static",
                    "config": {
                        "hasFilters": false,
                        "totalRow": false,
                        "firstColumn": true,
                        "lastColumn": false,
                        "numberOfHeaders": 1,
                        "bandedRows": true,
                        "bandedColumns": false,
                        "styleId": "TableStyleMedium5"
                    }
                },
                {
                    "range": "A1:F12",
                    "type": "static",
                    "config": {
                        "hasFilters": false,
                        "totalRow": false,
                        "firstColumn": true,
                        "lastColumn": false,
                        "numberOfHeaders": 1,
                        "bandedRows": true,
                        "bandedColumns": false,
                        "styleId": "TableStyleMedium5"
                    }
                }
            ],
            "areGridLinesVisible": true,
            "isVisible": true,
            "headerGroups": { "ROW": [], "COL": [] },
            "dataValidationRules": [],
            "comments": {}
        }
    ],
    "styles": {
        "1": { "fontSize": 14, "textColor": "#01666B", "bold": true },
        "2": { "fontSize": 16, "textColor": "#01666B", "bold": true },
        "3": { "textColor": "#434343", "bold": true, "fontSize": 11 },
        "4": { "verticalAlign": "middle", "textColor": "#434343" },
        "5": { "fontSize": 16 },
        "6": {
            "textColor": "#434343",
            "bold": true,
            "fontSize": 11,
            "align": "center"
        },
        "7": { "verticalAlign": "middle" },
        "8": { "bold": true, "fillColor": "#E6F2F3" },
        "9": { "fillColor": "#E6F2F3" }
    },
    "formats": { "1": "0.00%", "2": "#,##0.00", "3": "#,##0" },
    "borders": {
        "1": { "bottom": { "style": "thin", "color": "#CCCCCC" } },
        "2": { "top": { "style": "thin", "color": "#CCCCCC" } }
    },
    "revisionId": "ac18891f-8749-4112-b6bf-1c5d232845a1",
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
        "c14d538a-a794-4ea2-ab95-70089c764c85": {
            "type": "ODOO",
            "domain": [["location_id.usage", "=", "internal"]],
            "context": {
                "mail_notify_force_send": false,
                "always_show_loc": 1,
                "inventory_mode": true,
                "inventory_report_mode": true
            },
            "sortedColumn": {
                "groupId": [[], []],
                "measure": "quantity",
                "order": "desc",
                "originIndexes": [0]
            },
            "measures": [
                {
                    "id": "quantity",
                    "fieldName": "quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "reserved_quantity",
                    "fieldName": "reserved_quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "available_quantity",
                    "fieldName": "available_quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "inventory_quantity",
                    "fieldName": "inventory_quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "inventory_diff_quantity",
                    "fieldName": "inventory_diff_quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "cyclic_inventory_frequency",
                    "fieldName": "cyclic_inventory_frequency",
                    "aggregator": "sum"
                },
                {
                    "id": "inventory_quantity_auto_apply",
                    "fieldName": "inventory_quantity_auto_apply",
                    "aggregator": "sum"
                },
                { "id": "value", "fieldName": "value", "aggregator": "sum" },
                { "id": "__count", "fieldName": "__count" }
            ],
            "model": "stock.quant",
            "columns": [],
            "rows": [{ "fieldName": "location_id" }],
            "name": "Inventory - locations",
            "actionXmlId": "stock.dashboard_open_quants",
            "formulaId": "8",
            "fieldMatching": {
                "62f6d1bd-9c76-48d7-b95a-3f5d6cb505df": {
                    "chain": "location_id",
                    "type": "many2one"
                },
                "8dea5478-ff09-41fc-ada0-cf23c66e4f8b": {
                    "chain": "product_id",
                    "type": "many2one"
                },
                "0725cf6d-c980-46a1-bd7b-bd75e34348e7": {
                    "chain": "lot_id",
                    "type": "many2one"
                },
                "7ff87378-db1d-4725-bf25-6eee704b15a5": {
                    "chain": "product_categ_id",
                    "type": "many2one"
                },
                "feeb0d08-4596-4e7c-b478-a3c659189cdd": {
                    "chain": "warehouse_id",
                    "type": "many2one"
                }
            }
        },
        "4f0d7a37-943e-4289-ac4b-3cf1b56ca17a": {
            "type": "ODOO",
            "domain": [["location_id.usage", "=", "internal"]],
            "context": {
                "mail_notify_force_send": false,
                "always_show_loc": 1,
                "inventory_mode": true,
                "inventory_report_mode": true
            },
            "sortedColumn": {
                "groupId": [[], []],
                "measure": "quantity",
                "order": "desc",
                "originIndexes": [0]
            },
            "measures": [
                {
                    "id": "quantity",
                    "fieldName": "quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "reserved_quantity",
                    "fieldName": "reserved_quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "available_quantity",
                    "fieldName": "available_quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "inventory_quantity",
                    "fieldName": "inventory_quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "inventory_diff_quantity",
                    "fieldName": "inventory_diff_quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "cyclic_inventory_frequency",
                    "fieldName": "cyclic_inventory_frequency",
                    "aggregator": "sum"
                },
                {
                    "id": "inventory_quantity_auto_apply",
                    "fieldName": "inventory_quantity_auto_apply",
                    "aggregator": "sum"
                },
                { "id": "value", "fieldName": "value", "aggregator": "sum" },
                { "id": "__count", "fieldName": "__count" }
            ],
            "model": "stock.quant",
            "columns": [],
            "rows": [{ "fieldName": "product_id" }],
            "name": "Inventory - products sort by Qty",
            "actionXmlId": "stock.dashboard_open_quants",
            "formulaId": "9",
            "fieldMatching": {
                "62f6d1bd-9c76-48d7-b95a-3f5d6cb505df": {
                    "chain": "location_id",
                    "type": "many2one"
                },
                "8dea5478-ff09-41fc-ada0-cf23c66e4f8b": {
                    "chain": "product_id",
                    "type": "many2one"
                },
                "0725cf6d-c980-46a1-bd7b-bd75e34348e7": {
                    "chain": "lot_id",
                    "type": "many2one"
                },
                "7ff87378-db1d-4725-bf25-6eee704b15a5": {
                    "chain": "product_categ_id",
                    "type": "many2one"
                },
                "feeb0d08-4596-4e7c-b478-a3c659189cdd": {
                    "chain": "warehouse_id",
                    "type": "many2one"
                }
            }
        },
        "22f3a887-f193-4be2-a4af-e991b7ccc65e": {
            "type": "ODOO",
            "domain": [
                "&",
                ["location_id.usage", "=", "internal"],
                ["reserved_quantity", ">", 0]
            ],
            "context": {
                "mail_notify_force_send": false,
                "always_show_loc": 1,
                "inventory_mode": true,
                "inventory_report_mode": true
            },
            "sortedColumn": {
                "groupId": [[], []],
                "measure": "quantity",
                "order": "desc",
                "originIndexes": [0]
            },
            "measures": [
                {
                    "id": "quantity",
                    "fieldName": "quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "reserved_quantity",
                    "fieldName": "reserved_quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "available_quantity",
                    "fieldName": "available_quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "inventory_quantity",
                    "fieldName": "inventory_quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "inventory_diff_quantity",
                    "fieldName": "inventory_diff_quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "cyclic_inventory_frequency",
                    "fieldName": "cyclic_inventory_frequency",
                    "aggregator": "sum"
                },
                {
                    "id": "inventory_quantity_auto_apply",
                    "fieldName": "inventory_quantity_auto_apply",
                    "aggregator": "sum"
                },
                { "id": "value", "fieldName": "value", "aggregator": "sum" },
                { "id": "__count", "fieldName": "__count" }
            ],
            "model": "stock.quant",
            "columns": [],
            "rows": [{ "fieldName": "location_id" }],
            "name": "Inventory - locations (reserved stock)",
            "actionXmlId": "stock.dashboard_open_quants",
            "formulaId": "18",
            "fieldMatching": {
                "62f6d1bd-9c76-48d7-b95a-3f5d6cb505df": {
                    "chain": "location_id",
                    "type": "many2one"
                },
                "8dea5478-ff09-41fc-ada0-cf23c66e4f8b": {
                    "chain": "product_id",
                    "type": "many2one"
                },
                "0725cf6d-c980-46a1-bd7b-bd75e34348e7": {
                    "chain": "lot_id",
                    "type": "many2one"
                },
                "7ff87378-db1d-4725-bf25-6eee704b15a5": {
                    "chain": "product_categ_id",
                    "type": "many2one"
                },
                "feeb0d08-4596-4e7c-b478-a3c659189cdd": {
                    "chain": "warehouse_id",
                    "type": "many2one"
                }
            }
        },
        "858023a6-cd63-47d6-9bce-759b488c2b75": {
            "type": "ODOO",
            "domain": [["location_id.usage", "=", "internal"]],
            "context": {
                "mail_notify_force_send": false,
                "always_show_loc": 1,
                "inventory_mode": true,
                "inventory_report_mode": true
            },
            "sortedColumn": {
                "groupId": [[], []],
                "measure": "value",
                "order": "desc",
                "originIndexes": [0]
            },
            "measures": [
                { "id": "value", "fieldName": "value", "aggregator": "sum" },
                {
                    "id": "quantity",
                    "fieldName": "quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "reserved_quantity",
                    "fieldName": "reserved_quantity",
                    "aggregator": "sum"
                }
            ],
            "model": "stock.quant",
            "columns": [],
            "rows": [{ "fieldName": "product_id" }],
            "name": "Inventory by Product sort by value",
            "actionXmlId": "stock.dashboard_open_quants",
            "formulaId": "20",
            "fieldMatching": {
                "62f6d1bd-9c76-48d7-b95a-3f5d6cb505df": {
                    "chain": "location_id",
                    "type": "many2one"
                },
                "8dea5478-ff09-41fc-ada0-cf23c66e4f8b": {
                    "chain": "product_id",
                    "type": "many2one"
                },
                "0725cf6d-c980-46a1-bd7b-bd75e34348e7": {
                    "chain": "lot_id",
                    "type": "many2one"
                },
                "7ff87378-db1d-4725-bf25-6eee704b15a5": {
                    "chain": "product_categ_id",
                    "type": "many2one"
                },
                "feeb0d08-4596-4e7c-b478-a3c659189cdd": {
                    "chain": "warehouse_id",
                    "type": "many2one"
                }
            }
        },
        "37e97fb2-1885-4dbd-9632-ec5c54a776f4": {
            "type": "ODOO",
            "domain": [["location_id.usage", "=", "internal"]],
            "context": {
                "mail_notify_force_send": false,
                "always_show_loc": 1,
                "inventory_mode": true,
                "inventory_report_mode": true
            },
            "sortedColumn": {
                "groupId": [[], []],
                "measure": "value",
                "order": "desc",
                "originIndexes": [0]
            },
            "measures": [
                {
                    "id": "reserved_quantity",
                    "fieldName": "reserved_quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "quantity",
                    "fieldName": "quantity",
                    "aggregator": "sum"
                },
                { "id": "value", "fieldName": "value", "aggregator": "sum" }
            ],
            "model": "stock.quant",
            "columns": [],
            "rows": [{ "fieldName": "location_id" }],
            "name": "Inventory by Location sort by value",
            "actionXmlId": "stock.dashboard_open_quants",
            "formulaId": "21",
            "fieldMatching": {
                "62f6d1bd-9c76-48d7-b95a-3f5d6cb505df": {
                    "chain": "location_id",
                    "type": "many2one"
                },
                "8dea5478-ff09-41fc-ada0-cf23c66e4f8b": {
                    "chain": "product_id",
                    "type": "many2one"
                },
                "0725cf6d-c980-46a1-bd7b-bd75e34348e7": {
                    "chain": "lot_id",
                    "type": "many2one"
                },
                "7ff87378-db1d-4725-bf25-6eee704b15a5": {
                    "chain": "product_categ_id",
                    "type": "many2one"
                },
                "feeb0d08-4596-4e7c-b478-a3c659189cdd": {
                    "chain": "warehouse_id",
                    "type": "many2one"
                }
            }
        },
        "8f099f8d-362a-41e5-b676-75ab717cfc1e": {
            "type": "ODOO",
            "domain": [
                "&",
                ["location_id.usage", "=", "internal"],
                ["quantity", "<", 0]
            ],
            "context": {
                "mail_notify_force_send": false,
                "always_show_loc": 1,
                "inventory_mode": true,
                "inventory_report_mode": true
            },
            "sortedColumn": {
                "groupId": [[], [8]],
                "measure": "quantity",
                "order": "asc",
                "originIndexes": [0]
            },
            "measures": [
                {
                    "id": "quantity",
                    "fieldName": "quantity",
                    "aggregator": "sum"
                },
                { "id": "__count", "fieldName": "__count" }
            ],
            "model": "stock.quant",
            "columns": [],
            "rows": [{ "fieldName": "product_id" }],
            "name": "Inventory by Location",
            "actionXmlId": "stock.dashboard_open_quants",
            "formulaId": "22",
            "fieldMatching": {
                "feeb0d08-4596-4e7c-b478-a3c659189cdd": {
                    "chain": "warehouse_id",
                    "type": "many2one"
                },
                "62f6d1bd-9c76-48d7-b95a-3f5d6cb505df": {
                    "chain": "location_id",
                    "type": "many2one"
                },
                "7ff87378-db1d-4725-bf25-6eee704b15a5": {
                    "chain": "product_categ_id",
                    "type": "many2one"
                },
                "8dea5478-ff09-41fc-ada0-cf23c66e4f8b": {
                    "chain": "product_id",
                    "type": "many2one"
                },
                "0725cf6d-c980-46a1-bd7b-bd75e34348e7": {
                    "chain": "lot_id",
                    "type": "many2one"
                }
            }
        },
        "0885aadb-2384-4300-a531-893035a13173": {
            "type": "ODOO",
            "domain": [
                "&",
                ["location_id.usage", "=", "internal"],
                ["reserved_quantity", ">", 0]
            ],
            "context": {
                "mail_notify_force_send": false,
                "always_show_loc": 1,
                "inventory_mode": true,
                "inventory_report_mode": true
            },
            "sortedColumn": {
                "groupId": [[], []],
                "measure": "value",
                "order": "desc",
                "originIndexes": [0]
            },
            "measures": [
                {
                    "id": "reserved_quantity",
                    "fieldName": "reserved_quantity",
                    "aggregator": "sum"
                },
                {
                    "id": "quantity",
                    "fieldName": "quantity",
                    "aggregator": "sum"
                },
                { "id": "value", "fieldName": "value", "aggregator": "sum" }
            ],
            "model": "stock.quant",
            "columns": [],
            "rows": [{ "fieldName": "location_id" }],
            "name": "Inventory by Location sort by value (reserved stock)",
            "actionXmlId": "stock.dashboard_open_quants",
            "formulaId": "23",
            "fieldMatching": {
                "62f6d1bd-9c76-48d7-b95a-3f5d6cb505df": {
                    "chain": "location_id",
                    "type": "many2one"
                },
                "8dea5478-ff09-41fc-ada0-cf23c66e4f8b": {
                    "chain": "product_id",
                    "type": "many2one"
                },
                "0725cf6d-c980-46a1-bd7b-bd75e34348e7": {
                    "chain": "lot_id",
                    "type": "many2one"
                },
                "7ff87378-db1d-4725-bf25-6eee704b15a5": {
                    "chain": "product_categ_id",
                    "type": "many2one"
                },
                "feeb0d08-4596-4e7c-b478-a3c659189cdd": {
                    "chain": "warehouse_id",
                    "type": "many2one"
                }
            }
        }
    },
    "pivotNextId": 25,
    "customTableStyles": {},
    "odooVersion": 12,
    "globalFilters": [
        {
            "id": "feeb0d08-4596-4e7c-b478-a3c659189cdd",
            "type": "relation",
            "label": "Warehouse",
            "defaultValue": [],
            "defaultValueDisplayNames": [],
            "modelName": "stock.warehouse",
            "includeChildren": false
        },
        {
            "id": "62f6d1bd-9c76-48d7-b95a-3f5d6cb505df",
            "type": "relation",
            "label": "Location",
            "defaultValue": [],
            "defaultValueDisplayNames": [],
            "modelName": "stock.location"
        },
        {
            "id": "7ff87378-db1d-4725-bf25-6eee704b15a5",
            "type": "relation",
            "label": "Product Category",
            "defaultValue": [],
            "defaultValueDisplayNames": [],
            "modelName": "product.category",
            "includeChildren": true
        },
        {
            "id": "8dea5478-ff09-41fc-ada0-cf23c66e4f8b",
            "type": "relation",
            "label": "Product",
            "defaultValue": [],
            "defaultValueDisplayNames": [],
            "modelName": "product.product"
        },
        {
            "id": "0725cf6d-c980-46a1-bd7b-bd75e34348e7",
            "type": "relation",
            "label": "Lot/Serial",
            "modelName": "stock.lot",
            "defaultValue": [],
            "defaultValueDisplayNames": [],
            "rangeType": "year"
        }
    ],
    "lists": {},
    "listNextId": 3,
    "chartOdooMenusReferences": {
        "fad86b76-037f-4b7f-8afc-bc028c2536de": "stock.menu_valuation",
        "73f61914-d895-4484-b3ba-585107daeb44": "stock.menu_valuation",
        "de7010e1-2cdc-4e19-a1de-1c7fd4795bf4": "stock.menu_valuation",
        "833374d1-d09f-4a4e-bb09-2831ceee5465": "stock.menu_valuation",
        "8324b219-c649-4392-9fee-a3e7f396d70c": "stock.menu_valuation",
        "0d527983-ee0f-44cb-af82-f637574a4e1e": "stock.menu_valuation",
        "ebde3556-983a-475a-95f0-4edb2515b3bb": "stock.menu_valuation",
        "e7f90aaf-95ec-4f1d-9b0c-ec7e4465d772": "stock.menu_valuation",
        "639d221d-d74d-4762-984e-9debac4f5a82": "stock.menu_valuation"
    }
}

```

## File: data\files\warehouse_metrics_sample_dashboard.json

```json
{
    "version": 22,
    "sheets": [
        {
            "id": "sheet1",
            "name": "Dashboard",
            "colNumber": 8,
            "rowNumber": 72,
            "rows": {
                "6": {
                    "size": 38
                },
                "22": {
                    "size": 40
                },
                "23": {
                    "size": 21
                },
                "24": {
                    "size": 21
                },
                "25": {
                    "size": 21
                },
                "26": {
                    "size": 21
                },
                "27": {
                    "size": 21
                },
                "28": {
                    "size": 21
                },
                "29": {
                    "size": 21
                },
                "30": {
                    "size": 21
                },
                "31": {
                    "size": 21
                },
                "32": {
                    "size": 21
                },
                "33": {
                    "size": 21
                },
                "34": {
                    "size": 21
                },
                "35": {
                    "size": 21
                },
                "36": {
                    "size": 21
                },
                "37": {
                    "size": 21
                },
                "38": {
                    "size": 21
                },
                "39": {
                    "size": 21
                },
                "40": {
                    "size": 41
                },
                "41": {
                    "size": 21
                },
                "42": {
                    "size": 21
                },
                "43": {
                    "size": 21
                },
                "44": {
                    "size": 21
                },
                "45": {
                    "size": 21
                },
                "46": {
                    "size": 21
                },
                "47": {
                    "size": 21
                },
                "48": {
                    "size": 21
                },
                "49": {
                    "size": 21
                },
                "50": {
                    "size": 21
                },
                "51": {
                    "size": 21
                },
                "52": {
                    "size": 21
                },
                "53": {
                    "size": 21
                },
                "54": {
                    "size": 21
                },
                "55": {
                    "size": 21
                },
                "56": {
                    "size": 21
                },
                "57": {
                    "size": 36
                },
                "58": {
                    "size": 38
                },
                "59": {
                    "size": 27
                },
                "60": {
                    "size": 27
                },
                "61": {
                    "size": 27
                },
                "62": {
                    "size": 27
                },
                "63": {
                    "size": 27
                },
                "64": {
                    "size": 27
                },
                "65": {
                    "size": 27
                },
                "66": {
                    "size": 27
                },
                "67": {
                    "size": 27
                },
                "68": {
                    "size": 27
                },
                "69": {
                    "size": 21
                },
                "70": {
                    "size": 21
                },
                "71": {
                    "size": 21
                }
            },
            "cols": {
                "0": {
                    "size": 332
                },
                "1": {
                    "size": 100
                },
                "2": {
                    "size": 69
                },
                "3": {
                    "size": 40
                },
                "4": {
                    "size": 50
                },
                "5": {
                    "size": 275
                },
                "6": {
                    "size": 100
                },
                "7": {
                    "size": 95
                }
            },
            "merges": [],
            "cells": {
                "A7": {
                    "content": "=_t(\"Available and reserved stock qty (top locations)\")"
                },
                "A23": {
                    "content": "=_t(\"Available and reserved stock qty (top propducts)\")"
                },
                "A41": {
                    "content": "=_t(\"Ageing stock qty by category and creation date\")"
                },
                "A58": {
                    "content": "=_t(\"Top 10 products with negative stock\")"
                },
                "A59": {
                    "content": "=_t(\"Products\")"
                },
                "E7": {
                    "content": "=_t(\"Available and reserved stock value (top locations)\")"
                },
                "E23": {
                    "content": "=_t(\"Available and reserved stock value (top propducts)\")"
                },
                "E41": {
                    "content": "=_t(\"Ageing stock value by product and creation date\")"
                }
            },
            "styles": {
                "A7": 1,
                "A23": 1,
                "A41": 1,
                "A58": 1,
                "E7": 1,
                "E23": 1,
                "E41": 1,
                "A59": 2
            },
            "formats": {},
            "borders": {
                "A7:C7": 1,
                "A23:C23": 1,
                "A41:C41": 1,
                "A58:C58": 1,
                "E7:H7": 1,
                "E23:H23": 1,
                "E41:H41": 1,
                "A8:C8": 2,
                "A24:C24": 2,
                "A42:C42": 2,
                "A59:C59": 2,
                "E8:H8": 2,
                "E24:H24": 2,
                "E42:H42": 2
            },
            "conditionalFormats": [],
            "figures": [
                {
                    "id": "833374d1-d09f-4a4e-bb09-2831ceee5465",
                    "x": 246,
                    "y": 9,
                    "width": 237,
                    "height": 108,
                    "tag": "chart",
                    "data": {
                        "baselineColorDown": "#E06666",
                        "baselineColorUp": "#6AA84F",
                        "baselineMode": "text",
                        "title": {
                            "text": "Share reserved stock Value",
                            "align": "left",
                            "bold": true,
                            "color": "#434343"
                        },
                        "type": "scorecard",
                        "background": "#ECFDF5",
                        "baseline": "Data!E4",
                        "keyValue": "Data!B4",
                        "humanize": false
                    }
                },
                {
                    "id": "de7010e1-2cdc-4e19-a1de-1c7fd4795bf4",
                    "x": 0,
                    "y": 9,
                    "width": 237,
                    "height": 108,
                    "tag": "chart",
                    "data": {
                        "baselineColorDown": "#E06666",
                        "baselineColorUp": "#6AA84F",
                        "baselineMode": "text",
                        "title": {
                            "text": "Share reserved stock Qty",
                            "align": "left",
                            "bold": true,
                            "color": "#434343"
                        },
                        "type": "scorecard",
                        "background": "#ECFDF5",
                        "baseline": "Data!E3",
                        "keyValue": "Data!B3",
                        "humanize": false
                    }
                },
                {
                    "id": "639d221d-d74d-4762-984e-9debac4f5a82",
                    "x": 492,
                    "y": 9,
                    "width": 237,
                    "height": 108,
                    "tag": "chart",
                    "data": {
                        "baselineColorDown": "#E06666",
                        "baselineColorUp": "#6AA84F",
                        "baselineMode": "difference",
                        "title": {
                            "text": "Lines with negative stock",
                            "bold": true,
                            "align": "left",
                            "color": "#434343"
                        },
                        "type": "scorecard",
                        "background": "#EFF6FF",
                        "keyValue": "Data!B5",
                        "humanize": true
                    }
                },
                {
                    "id": "c6b5d23a-bd70-4c85-b5d8-174914e392cf",
                    "x": 0,
                    "y": 176,
                    "width": 501,
                    "height": 344,
                    "tag": "chart",
                    "data": {
                        "type": "bar",
                        "dataSetsHaveTitle": true,
                        "dataSets": [
                            {
                                "dataRange": "Data!B11:B17",
                                "yAxisId": "y"
                            },
                            {
                                "dataRange": "Data!C11:C17"
                            }
                        ],
                        "legendPosition": "top",
                        "labelRange": "Data!A11:A17",
                        "title": {},
                        "stacked": true,
                        "aggregated": false,
                        "horizontal": false
                    }
                },
                {
                    "id": "0583c3a5-77a3-40e9-a8ae-a80c4923cdd8",
                    "x": 540,
                    "y": 176,
                    "width": 481,
                    "height": 344,
                    "tag": "chart",
                    "data": {
                        "type": "bar",
                        "dataSetsHaveTitle": true,
                        "dataSets": [
                            {
                                "dataRange": "Data!B20:B25",
                                "yAxisId": "y"
                            },
                            {
                                "dataRange": "Data!C20:C25"
                            }
                        ],
                        "legendPosition": "top",
                        "labelRange": "Data!A20:A25",
                        "title": {},
                        "stacked": true,
                        "aggregated": false,
                        "horizontal": false
                    }
                },
                {
                    "id": "19195543-e12e-40e4-b404-e1d27d0629a9",
                    "x": 0,
                    "y": 561,
                    "width": 501,
                    "height": 358,
                    "tag": "chart",
                    "data": {
                        "type": "bar",
                        "dataSetsHaveTitle": true,
                        "dataSets": [
                            {
                                "dataRange": "Data!B28:B36",
                                "yAxisId": "y"
                            },
                            {
                                "dataRange": "Data!C28:C36"
                            }
                        ],
                        "legendPosition": "top",
                        "labelRange": "Data!A28:A36",
                        "title": {},
                        "stacked": true,
                        "aggregated": false,
                        "horizontal": false
                    }
                },
                {
                    "id": "4abd0d9a-4754-4197-bf1d-8580e5676f31",
                    "x": 541,
                    "y": 561,
                    "width": 480,
                    "height": 356,
                    "tag": "chart",
                    "data": {
                        "type": "bar",
                        "dataSetsHaveTitle": true,
                        "dataSets": [
                            {
                                "dataRange": "Data!B39:B47",
                                "yAxisId": "y"
                            },
                            {
                                "dataRange": "Data!C39:C47"
                            }
                        ],
                        "legendPosition": "top",
                        "labelRange": "Data!A39:A47",
                        "title": {},
                        "stacked": true,
                        "aggregated": false,
                        "horizontal": false
                    }
                },
                {
                    "id": "8ed2d718-6948-4dc5-aa0a-034c5ec345d1",
                    "x": 0,
                    "y": 959,
                    "width": 503,
                    "height": 335,
                    "tag": "chart",
                    "data": {
                        "type": "bar",
                        "dataSetsHaveTitle": false,
                        "dataSets": [
                            {
                                "dataRange": "Data!B50:B52",
                                "yAxisId": "y"
                            },
                            {
                                "dataRange": "Data!C50:C52"
                            },
                            {
                                "dataRange": "Data!D50:D52"
                            },
                            {
                                "dataRange": "Data!E50:E52"
                            },
                            {
                                "dataRange": "Data!F50:F52"
                            }
                        ],
                        "legendPosition": "none",
                        "labelRange": "Data!A50:A52",
                        "title": {},
                        "stacked": true,
                        "aggregated": false,
                        "horizontal": false
                    }
                },
                {
                    "id": "fdbb3758-85b5-4d59-b8bf-621b65e0c7b6",
                    "x": 540,
                    "y": 959,
                    "width": 481,
                    "height": 335,
                    "tag": "chart",
                    "data": {
                        "type": "bar",
                        "dataSetsHaveTitle": false,
                        "dataSets": [
                            {
                                "dataRange": "Data!B50:B52",
                                "yAxisId": "y"
                            },
                            {
                                "dataRange": "Data!C50:C52"
                            },
                            {
                                "dataRange": "Data!D50:D52"
                            },
                            {
                                "dataRange": "Data!E50:E52"
                            },
                            {
                                "dataRange": "Data!F50:F52"
                            }
                        ],
                        "legendPosition": "none",
                        "labelRange": "Data!A50:A52",
                        "title": {},
                        "stacked": true,
                        "aggregated": false,
                        "horizontal": false
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
            "id": "fb6d5d91-04cf-4c22-953a-a00c4e8f19e4",
            "name": "Data",
            "colNumber": 22,
            "rowNumber": 82,
            "rows": {},
            "cols": {},
            "merges": [],
            "cells": {
                "A1": {
                    "content": "=_t(\"KPI\")"
                },
                "A2": {
                    "content": "=_t(\"Total inventory value\")"
                },
                "A3": {
                    "content": "=_t(\"Share of reserved stock qty\")"
                },
                "A4": {
                    "content": "=_t(\"Share of reserved stock Value\")"
                },
                "A5": {
                    "content": "=_t(\"Count of products with negative stock\")"
                },
                "A12": {
                    "content": "=_t(\"WH/Stock\")"
                },
                "A13": {
                    "content": "=_t(\"WH/Output\")"
                },
                "A14": {
                    "content": "=_t(\"Pre-production\")"
                },
                "A15": {
                    "content": "=_t(\"Post-production\")"
                },
                "A16": {
                    "content": "=_t(\"WH/Stock/Shelf 10\")"
                },
                "A21": {
                    "content": "=_t(\"WH/Stock\")"
                },
                "A22": {
                    "content": "=_t(\"WH/Output\")"
                },
                "A23": {
                    "content": "=_t(\"Pre-production\")"
                },
                "A24": {
                    "content": "=_t(\"Post-production\")"
                },
                "A25": {
                    "content": "=_t(\"WH/Stock/Shelf 10\")"
                },
                "A29": {
                    "content": "=_t(\"Electric standing desk\")"
                },
                "A30": {
                    "content": "=_t(\"Smart air purifier\")"
                },
                "A31": {
                    "content": "=_t(\"Waterproof hiking backpack\")"
                },
                "A32": {
                    "content": "=_t(\"Solar-powered phone charger\")"
                },
                "A33": {
                    "content": "=_t(\"3D printing pen\")"
                },
                "A34": {
                    "content": "=_t(\"Compact espresso machine\")"
                },
                "A35": {
                    "content": "=_t(\"Bluetooth-enabled LED light strip\")"
                },
                "A36": {
                    "content": "=_t(\"Ergonomic office chair\")"
                },
                "A40": {
                    "content": "=_t(\"Electric standing desk\")"
                },
                "A41": {
                    "content": "=_t(\"Smart air purifier\")"
                },
                "A42": {
                    "content": "=_t(\"Waterproof hiking backpack\")"
                },
                "A43": {
                    "content": "=_t(\"Solar-powered phone charger\")"
                },
                "A44": {
                    "content": "=_t(\"3D printing pen\")"
                },
                "A45": {
                    "content": "=_t(\"Compact espresso machine\")"
                },
                "A46": {
                    "content": "=_t(\"Bluetooth-enabled LED light strip\")"
                },
                "A47": {
                    "content": "=_t(\"Ergonomic office chair\")"
                },
                "A50": {
                    "content": "=EDATE(TODAY(),-6)"
                },
                "A51": {
                    "content": "=EDATE(TODAY(),-3)"
                },
                "A52": {
                    "content": "=EDATE(TODAY(),0)"
                },
                "A55": {
                    "content": "=EDATE(TODAY(),-6)"
                },
                "A56": {
                    "content": "=EDATE(TODAY(),-3)"
                },
                "A57": {
                    "content": "=EDATE(TODAY(),0)"
                },
                "B2": {
                    "content": "188071"
                },
                "B3": {
                    "content": "0.2408405172413793"
                },
                "B4": {
                    "content": "0.4064262964518719"
                },
                "B5": {
                    "content": "6"
                },
                "B11": {
                    "content": "=_t(\"Available Quantity\")"
                },
                "B12": {
                    "content": "3377"
                },
                "B13": {
                    "content": "598"
                },
                "B14": {
                    "content": "3826"
                },
                "B15": {
                    "content": "2772"
                },
                "B16": {
                    "content": "3455"
                },
                "B20": {
                    "content": "=_t(\"Available Value\")"
                },
                "B21": {
                    "content": "34189"
                },
                "B22": {
                    "content": "31472"
                },
                "B23": {
                    "content": "48745"
                },
                "B24": {
                    "content": "41379"
                },
                "B25": {
                    "content": "29347"
                },
                "B28": {
                    "content": "=_t(\"Available Quantity\")"
                },
                "B29": {
                    "content": "55"
                },
                "B30": {
                    "content": "43"
                },
                "B31": {
                    "content": "32"
                },
                "B32": {
                    "content": "57"
                },
                "B33": {
                    "content": "71"
                },
                "B34": {
                    "content": "20"
                },
                "B35": {
                    "content": "33"
                },
                "B36": {
                    "content": "11"
                },
                "B39": {
                    "content": "=_t(\"Available Value\")"
                },
                "B40": {
                    "content": "1986"
                },
                "B41": {
                    "content": "6388"
                },
                "B42": {
                    "content": "7098"
                },
                "B43": {
                    "content": "5878"
                },
                "B44": {
                    "content": "7870"
                },
                "B45": {
                    "content": "3064"
                },
                "B46": {
                    "content": "5372"
                },
                "B47": {
                    "content": "5213"
                },
                "B50": {
                    "content": "483"
                },
                "B51": {
                    "content": "108"
                },
                "B52": {
                    "content": "236"
                },
                "B55": {
                    "content": "88"
                },
                "B56": {
                    "content": "403"
                },
                "B57": {
                    "content": "119"
                },
                "C1": {
                    "content": "=_t(\"Reserved\")"
                },
                "C3": {
                    "content": "447"
                },
                "C4": {
                    "content": "76437"
                },
                "C11": {
                    "content": "=_t(\"Reserved Quantity\")"
                },
                "C12": {
                    "content": "4483"
                },
                "C13": {
                    "content": "4782"
                },
                "C14": {
                    "content": "4603"
                },
                "C15": {
                    "content": "2226"
                },
                "C16": {
                    "content": "1345"
                },
                "C20": {
                    "content": "=_t(\"Reserved Value\")"
                },
                "C21": {
                    "content": "44891"
                },
                "C22": {
                    "content": "48745"
                },
                "C23": {
                    "content": "29347"
                },
                "C24": {
                    "content": "38686"
                },
                "C25": {
                    "content": "48745"
                },
                "C28": {
                    "content": "=_t(\"Reserved Quantity\")"
                },
                "C29": {
                    "content": "69"
                },
                "C30": {
                    "content": "15"
                },
                "C31": {
                    "content": "55"
                },
                "C32": {
                    "content": "72"
                },
                "C33": {
                    "content": "62"
                },
                "C34": {
                    "content": "40"
                },
                "C35": {
                    "content": "38"
                },
                "C36": {
                    "content": "30"
                },
                "C39": {
                    "content": "=_t(\"Reserved Value\")"
                },
                "C40": {
                    "content": "2313"
                },
                "C41": {
                    "content": "3376"
                },
                "C42": {
                    "content": "1124"
                },
                "C43": {
                    "content": "4626"
                },
                "C44": {
                    "content": "4886"
                },
                "C45": {
                    "content": "7053"
                },
                "C46": {
                    "content": "4888"
                },
                "C47": {
                    "content": "6652"
                },
                "C50": {
                    "content": "337"
                },
                "C51": {
                    "content": "285"
                },
                "C52": {
                    "content": "275"
                },
                "C55": {
                    "content": "119"
                },
                "C56": {
                    "content": "54"
                },
                "C57": {
                    "content": "310"
                },
                "D1": {
                    "content": "=_t(\"Total\")"
                },
                "D3": {
                    "content": "1856"
                },
                "D4": {
                    "content": "188071"
                },
                "D50": {
                    "content": "333"
                },
                "D51": {
                    "content": "215"
                },
                "D52": {
                    "content": "358"
                },
                "D55": {
                    "content": "91"
                },
                "D56": {
                    "content": "439"
                },
                "D57": {
                    "content": "319"
                },
                "E3": {
                    "content": "=_t(\"447 out of 1,856\")"
                },
                "E4": {
                    "content": "=_t(\"76,437 out of 188,071\")"
                },
                "E50": {
                    "content": "213"
                },
                "E51": {
                    "content": "247"
                },
                "E52": {
                    "content": "378"
                },
                "E55": {
                    "content": "97"
                },
                "E56": {
                    "content": "235"
                },
                "E57": {
                    "content": "227"
                },
                "F50": {
                    "content": "331"
                },
                "F51": {
                    "content": "373"
                },
                "F52": {
                    "content": "356"
                },
                "F55": {
                    "content": "67"
                },
                "F56": {
                    "content": "60"
                },
                "F57": {
                    "content": "135"
                }
            },
            "styles": {
                "A1": 3,
                "C1:D1": 3,
                "A2:A5": 4,
                "B1": 4
            },
            "formats": {
                "A50:A52": 1,
                "A55:A57": 1,
                "B2": 2,
                "D4": 2,
                "B3:B4": 3,
                "B5": 4,
                "C3:D3": 4,
                "B21:C25": 5,
                "B40:C47": 5,
                "C4": 6
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
            "fontSize": 16,
            "textColor": "#01666B",
            "bold": true
        },
        "2": {
            "textColor": "#434343",
            "bold": true,
            "fontSize": 11
        },
        "3": {
            "bold": true,
            "fillColor": "#E6F2F3"
        },
        "4": {
            "fillColor": "#E6F2F3"
        }
    },
    "formats": {
        "1": "qq yyyy",
        "2": "[$$]#,##0,[$k]",
        "3": "0.00%",
        "4": "#,##0.00",
        "5": "[$$]#,##0",
        "6": "[$$]#,##0[$]"
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
    "pivotNextId": 25,
    "customTableStyles": {},
    "odooVersion": 12,
    "globalFilters": [],
    "lists": {},
    "listNextId": 3
}

```

