# Odoo Module: spreadsheet_dashboard_sale

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
    'name': "Spreadsheet dashboard for sales",
    'version': '1.0',
    'category': 'Hidden',
    'summary': 'Spreadsheet',
    'description': 'Spreadsheet',
    'depends': ['spreadsheet_dashboard', 'sale'],
    'data': [
        "data/dashboards.xml",
    ],
    'auto_install': ['sale'],
    'license': 'LGPL-3',
}

```

## File: data\dashboards.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="spreadsheet_dashboard_sales" model="spreadsheet.dashboard">
        <field name="name">Sales</field>
        <field name="spreadsheet_binary_data" type="base64" file="spreadsheet_dashboard_sale/data/files/sales_dashboard.json"/>
        <field name="main_data_model_ids" eval="[(4, ref('sale.model_sale_order'))]"/>
        <field name="sample_dashboard_file_path">spreadsheet_dashboard_sale/data/files/sales_sample_dashboard.json</field>
        <field name="dashboard_group_id" ref="spreadsheet_dashboard.spreadsheet_dashboard_group_sales"/>
        <field name="group_ids" eval="[Command.link(ref('sales_team.group_sale_manager'))]"/>
        <field name="sequence">100</field>
        <field name="is_published">True</field>
    </record>

    <record id="spreadsheet_dashboard_product" model="spreadsheet.dashboard">
        <field name="name">Product</field>
        <field name="spreadsheet_binary_data" type="base64" file="spreadsheet_dashboard_sale/data/files/product_dashboard.json"/>
        <field name="main_data_model_ids" eval="[(4, ref('sale.model_sale_order'))]"/>
        <field name="sample_dashboard_file_path">spreadsheet_dashboard_sale/data/files/product_sample_dashboard.json</field>
        <field name="dashboard_group_id" ref="spreadsheet_dashboard.spreadsheet_dashboard_group_sales"/>
        <field name="group_ids" eval="[Command.link(ref('sales_team.group_sale_manager'))]"/>
        <field name="sequence">200</field>
        <field name="is_published">True</field>
    </record>

</odoo>

```

## File: data\files\product_dashboard.json

```json
{
  "version": 21,
  "sheets": [
    {
      "id": "sheet1",
      "name": "Dashboard",
      "colNumber": 7,
      "rowNumber": 69,
      "rows": {
        "6": { "size": 40 },
        "26": { "size": 40 },
        "46": { "size": 40 },
        "47": { "size": 40 },
        "48": { "size": 29 },
        "49": { "size": 29 },
        "50": { "size": 29 },
        "51": { "size": 29 },
        "52": { "size": 29 },
        "53": { "size": 29 },
        "54": { "size": 29 },
        "55": { "size": 29 },
        "56": { "size": 29 },
        "57": { "size": 29 }
      },
      "cols": {
        "0": { "size": 275 },
        "1": { "size": 100 },
        "2": { "size": 100 },
        "3": { "size": 50 },
        "4": { "size": 275 },
        "5": { "size": 100 },
        "6": { "size": 100 }
      },
      "merges": [],
      "cells": {
        "A7": {
          "content": "[Best Sellers by Revenue](odoo://view/{\"viewType\":\"graph\",\"action\":{\"domain\":[[\"state\", \"not in\", [\"draft\", \"sent\", \"cancel\"]]],\"context\":{\"group_by\":[\"product_id\"],\"graph_measure\":\"price_subtotal\",\"graph_mode\":\"bar\",\"graph_groupbys\":[\"product_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Best Sellers by Revenue\"})"
        },
        "A27": {
          "content": "[Best Sellers by Units Sold](odoo://view/{\"viewType\":\"graph\",\"action\":{\"domain\":[[\"state\", \"not in\", [\"draft\", \"sent\", \"cancel\"]]],\"context\":{\"group_by\":[\"product_id\"],\"graph_measure\":\"__count\",\"graph_mode\":\"bar\",\"graph_groupbys\":[\"product_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
        },
        "A47": {
          "content": "[Best Selling Products](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"state\", \"not in\", [\"draft\", \"sent\", \"cancel\"]]],\"context\":{\"group_by\":[\"product_id\"],\"pivot_measures\":[\"__count\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"product_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
        },
        "A48": { "content": "=_t(\"Product\")" },
        "A49": { "content": "=PIVOT.HEADER(1,\"#product_id\",1)" },
        "A50": { "content": "=PIVOT.HEADER(1,\"#product_id\",2)" },
        "A51": { "content": "=PIVOT.HEADER(1,\"#product_id\",3)" },
        "A52": { "content": "=PIVOT.HEADER(1,\"#product_id\",4)" },
        "A53": { "content": "=PIVOT.HEADER(1,\"#product_id\",5)" },
        "A54": { "content": "=PIVOT.HEADER(1,\"#product_id\",6)" },
        "A55": { "content": "=PIVOT.HEADER(1,\"#product_id\",7)" },
        "A56": { "content": "=PIVOT.HEADER(1,\"#product_id\",8)" },
        "A57": { "content": "=PIVOT.HEADER(1,\"#product_id\",9)" },
        "A58": { "content": "=PIVOT.HEADER(1,\"#product_id\",10)" },
        "B48": { "content": "=_t(\"Units\")" },
        "B49": {
          "content": "=PIVOT.VALUE(1,\"product_uom_qty\",\"#product_id\",1)"
        },
        "B50": {
          "content": "=PIVOT.VALUE(1,\"product_uom_qty\",\"#product_id\",2)"
        },
        "B51": {
          "content": "=PIVOT.VALUE(1,\"product_uom_qty\",\"#product_id\",3)"
        },
        "B52": {
          "content": "=PIVOT.VALUE(1,\"product_uom_qty\",\"#product_id\",4)"
        },
        "B53": {
          "content": "=PIVOT.VALUE(1,\"product_uom_qty\",\"#product_id\",5)"
        },
        "B54": {
          "content": "=PIVOT.VALUE(1,\"product_uom_qty\",\"#product_id\",6)"
        },
        "B55": {
          "content": "=PIVOT.VALUE(1,\"product_uom_qty\",\"#product_id\",7)"
        },
        "B56": {
          "content": "=PIVOT.VALUE(1,\"product_uom_qty\",\"#product_id\",8)"
        },
        "B57": {
          "content": "=PIVOT.VALUE(1,\"product_uom_qty\",\"#product_id\",9)"
        },
        "B58": {
          "content": "=PIVOT.VALUE(1,\"product_uom_qty\",\"#product_id\",10)"
        },
        "C48": { "content": "=_t(\"Revenue\")" },
        "C49": {
          "content": "=PIVOT.VALUE(1,\"price_subtotal\",\"#product_id\",1)"
        },
        "C50": {
          "content": "=PIVOT.VALUE(1,\"price_subtotal\",\"#product_id\",2)"
        },
        "C51": {
          "content": "=PIVOT.VALUE(1,\"price_subtotal\",\"#product_id\",3)"
        },
        "C52": {
          "content": "=PIVOT.VALUE(1,\"price_subtotal\",\"#product_id\",4)"
        },
        "C53": {
          "content": "=PIVOT.VALUE(1,\"price_subtotal\",\"#product_id\",5)"
        },
        "C54": {
          "content": "=PIVOT.VALUE(1,\"price_subtotal\",\"#product_id\",6)"
        },
        "C55": {
          "content": "=PIVOT.VALUE(1,\"price_subtotal\",\"#product_id\",7)"
        },
        "C56": {
          "content": "=PIVOT.VALUE(1,\"price_subtotal\",\"#product_id\",8)"
        },
        "C57": {
          "content": "=PIVOT.VALUE(1,\"price_subtotal\",\"#product_id\",9)"
        },
        "C58": {
          "content": "=PIVOT.VALUE(1,\"price_subtotal\",\"#product_id\",10)"
        },
        "E47": {
          "content": "[Best Selling Categories](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"state\", \"not in\", [\"draft\", \"sent\", \"cancel\"]]],\"context\":{\"group_by\":[\"categ_id\"],\"pivot_measures\":[\"__count\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"categ_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
        },
        "E48": { "content": "=_t(\"Category\")" },
        "E49": { "content": "=PIVOT.HEADER(2,\"#categ_id\",1)" },
        "E50": { "content": "=PIVOT.HEADER(2,\"#categ_id\",2)" },
        "E51": { "content": "=PIVOT.HEADER(2,\"#categ_id\",3)" },
        "E52": { "content": "=PIVOT.HEADER(2,\"#categ_id\",4)" },
        "E53": { "content": "=PIVOT.HEADER(2,\"#categ_id\",5)" },
        "E54": { "content": "=PIVOT.HEADER(2,\"#categ_id\",6)" },
        "E55": { "content": "=PIVOT.HEADER(2,\"#categ_id\",7)" },
        "E56": { "content": "=PIVOT.HEADER(2,\"#categ_id\",8)" },
        "E57": { "content": "=PIVOT.HEADER(2,\"#categ_id\",9)" },
        "E58": { "content": "=PIVOT.HEADER(2,\"#categ_id\",10)" },
        "F48": { "content": "=_t(\"Units\")" },
        "F49": {
          "content": "=PIVOT.VALUE(2,\"product_uom_qty\",\"#categ_id\",1)"
        },
        "F50": {
          "content": "=PIVOT.VALUE(2,\"product_uom_qty\",\"#categ_id\",2)"
        },
        "F51": {
          "content": "=PIVOT.VALUE(2,\"product_uom_qty\",\"#categ_id\",3)"
        },
        "F52": {
          "content": "=PIVOT.VALUE(2,\"product_uom_qty\",\"#categ_id\",4)"
        },
        "F53": {
          "content": "=PIVOT.VALUE(2,\"product_uom_qty\",\"#categ_id\",5)"
        },
        "F54": {
          "content": "=PIVOT.VALUE(2,\"product_uom_qty\",\"#categ_id\",6)"
        },
        "F55": {
          "content": "=PIVOT.VALUE(2,\"product_uom_qty\",\"#categ_id\",7)"
        },
        "F56": {
          "content": "=PIVOT.VALUE(2,\"product_uom_qty\",\"#categ_id\",8)"
        },
        "F57": {
          "content": "=PIVOT.VALUE(2,\"product_uom_qty\",\"#categ_id\",9)"
        },
        "F58": {
          "content": "=PIVOT.VALUE(2,\"product_uom_qty\",\"#categ_id\",10)"
        },
        "G48": { "content": "=_t(\"Revenue\")" },
        "G49": {
          "content": "=PIVOT.VALUE(2,\"price_subtotal\",\"#categ_id\",1)"
        },
        "G50": {
          "content": "=PIVOT.VALUE(2,\"price_subtotal\",\"#categ_id\",2)"
        },
        "G51": {
          "content": "=PIVOT.VALUE(2,\"price_subtotal\",\"#categ_id\",3)"
        },
        "G52": {
          "content": "=PIVOT.VALUE(2,\"price_subtotal\",\"#categ_id\",4)"
        },
        "G53": {
          "content": "=PIVOT.VALUE(2,\"price_subtotal\",\"#categ_id\",5)"
        },
        "G54": {
          "content": "=PIVOT.VALUE(2,\"price_subtotal\",\"#categ_id\",6)"
        },
        "G55": {
          "content": "=PIVOT.VALUE(2,\"price_subtotal\",\"#categ_id\",7)"
        },
        "G56": {
          "content": "=PIVOT.VALUE(2,\"price_subtotal\",\"#categ_id\",8)"
        },
        "G57": {
          "content": "=PIVOT.VALUE(2,\"price_subtotal\",\"#categ_id\",9)"
        },
        "G58": {
          "content": "=PIVOT.VALUE(2,\"price_subtotal\",\"#categ_id\",10)"
        }
      },
      "styles": {
        "A7": 1,
        "A27": 1,
        "A47": 1,
        "E47": 1,
        "A48": 2,
        "E48": 2,
        "A49:C58": 3,
        "E49:G58": 3,
        "B48:C48": 4,
        "F48:G48": 4
      },
      "formats": {},
      "borders": {
        "A47:C47": 1,
        "A7:G7": 1,
        "A27:G27": 1,
        "E47:G47": 1,
        "A48:C48": 2,
        "A8:G8": 2,
        "A28:G28": 2,
        "E48:G48": 2,
        "A49": 3,
        "E49": 3,
        "A50:A58": 4,
        "E50:E58": 4,
        "A59:C59": 5,
        "E59:G59": 5,
        "B49": 6,
        "F49": 6,
        "B50:B58": 7,
        "F50:F58": 7,
        "C49": 8,
        "G49": 8,
        "C50:C58": 9,
        "G50:G58": 9
      },
      "conditionalFormats": [
        {
          "rule": {
            "type": "DataBarRule",
            "color": 15531509,
            "rangeValues": "C49:C58"
          },
          "id": "3056558c-1b57-47bf-9c5f-6261b292e428",
          "ranges": ["A49:A58"]
        },
        {
          "rule": {
            "type": "DataBarRule",
            "color": 16775149,
            "rangeValues": "G49:G58"
          },
          "id": "136a8cad-572a-41e9-9cbe-e2d5b8d8dfa9",
          "ranges": ["E49:E58"]
        }
      ],
      "figures": [
        {
          "id": "e35856cf-9090-489b-b055-2d441380d954",
          "x": 0,
          "y": 178,
          "width": 1000,
          "height": 436,
          "tag": "chart",
          "data": {
            "title": { "text": "" },
            "background": "#FFFFFF",
            "legendPosition": "none",
            "metaData": {
              "groupBy": ["product_id"],
              "measure": "price_subtotal",
              "order": "DESC",
              "resModel": "sale.report",
              "mode": "bar"
            },
            "searchParams": {
              "comparison": null,
              "context": { "group_by": [] },
              "domain": [["state", "not in", ["draft", "sent", "cancel"]]],
              "groupBy": ["product_id"],
              "orderBy": []
            },
            "type": "odoo_bar",
            "verticalAxisPosition": "left",
            "stacked": true,
            "fieldMatching": {
              "a6c3274b-e53c-4c6b-90a8-bc3e3d109f52": {
                "chain": "date",
                "type": "datetime",
                "offset": 0
              },
              "36814ad9-adac-4aba-a5ad-df595a306ef7": {
                "chain": "product_id",
                "type": "many2one"
              },
              "edaf4d0c-df4b-48bc-b61a-48a07e8afd91": {
                "chain": "categ_id",
                "type": "many2one"
              }
            }
          }
        },
        {
          "id": "d0171069-d2cd-4c2c-a686-cd7515e93bb5",
          "x": 0,
          "y": 655,
          "width": 1000,
          "height": 438,
          "tag": "chart",
          "data": {
            "title": { "text": "" },
            "background": "#FFFFFF",
            "legendPosition": "none",
            "metaData": {
              "groupBy": ["product_id"],
              "measure": "product_uom_qty",
              "order": "DESC",
              "resModel": "sale.report",
              "mode": "bar"
            },
            "searchParams": {
              "comparison": null,
              "context": { "group_by": [] },
              "domain": [["state", "not in", ["draft", "sent", "cancel"]]],
              "groupBy": ["product_id"],
              "orderBy": []
            },
            "type": "odoo_bar",
            "verticalAxisPosition": "left",
            "stacked": true,
            "fieldMatching": {
              "a6c3274b-e53c-4c6b-90a8-bc3e3d109f52": {
                "chain": "date",
                "type": "datetime",
                "offset": 0
              },
              "36814ad9-adac-4aba-a5ad-df595a306ef7": {
                "chain": "product_id",
                "type": "many2one"
              },
              "edaf4d0c-df4b-48bc-b61a-48a07e8afd91": {
                "chain": "categ_id",
                "type": "many2one"
              }
            }
          }
        },
        {
          "id": "5f383918-4073-4f19-9cc9-603216c953ad",
          "x": 0,
          "y": 12,
          "width": 450,
          "height": 108,
          "tag": "chart",
          "data": {
            "baselineColorDown": "#DC6965",
            "baselineColorUp": "#00A04A",
            "baselineMode": "text",
            "title": {
              "text": "Best Seller",
              "color": "#434343",
              "bold": true
            },
            "type": "scorecard",
            "background": "#FEF2F2",
            "baseline": "Data!C2",
            "baselineDescr": "sold",
            "keyValue": "Data!B2",
            "humanize": false
          }
        },
        {
          "id": "ec57f69b-f2b1-4dfc-990c-91ab61b526bf",
          "x": 459,
          "y": 12,
          "width": 450,
          "height": 108,
          "tag": "chart",
          "data": {
            "baselineColorDown": "#DC6965",
            "baselineColorUp": "#00A04A",
            "baselineMode": "text",
            "title": {
              "text": "Best Category",
              "color": "#434343",
              "bold": true
            },
            "type": "scorecard",
            "background": "#FEF2F2",
            "baseline": "Data!C3",
            "baselineDescr": "sold",
            "keyValue": "Data!B3",
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
      "id": "a83a78f2-b124-4d6f-9726-125e62a32b8d",
      "name": "Data",
      "colNumber": 23,
      "rowNumber": 88,
      "rows": {},
      "cols": {},
      "merges": [],
      "cells": {
        "A1": { "content": "=_t(\"KPI\")" },
        "A2": { "content": "=_t(\"Best selling product\")" },
        "A3": { "content": "=_t(\"Best selling category\")" },
        "B1": { "content": "=_t(\"Name\")" },
        "B2": { "content": "=PIVOT.HEADER(1,\"#product_id\",1)" },
        "B3": { "content": "=PIVOT.HEADER(2,\"#categ_id\",1)" },
        "C1": { "content": "=_t(\"Units\")" },
        "C2": {
          "content": "=PIVOT.VALUE(1,\"product_uom_qty\",\"#product_id\",1)"
        },
        "C3": {
          "content": "=PIVOT.VALUE(2,\"product_uom_qty\",\"#categ_id\",1)"
        },
        "D1": { "content": "=_t(\"Revenue\")" },
        "D2": {
          "content": "=FORMAT.LARGE.NUMBER(PIVOT.VALUE(1,\"price_subtotal\",\"#product_id\",1))"
        },
        "D3": {
          "content": "=FORMAT.LARGE.NUMBER(PIVOT.VALUE(2,\"price_subtotal\",\"#categ_id\",1))"
        }
      },
      "styles": { "A1:A3": 5, "B1": 5, "C1:D3": 5, "B2:B3": 6 },
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
    "1": { "textColor": "#01666b", "bold": true, "fontSize": 16 },
    "2": { "textColor": "#434343", "fontSize": 11, "bold": true },
    "3": { "textColor": "#434343", "verticalAlign": "middle" },
    "4": {
      "textColor": "#434343",
      "fontSize": 11,
      "bold": true,
      "align": "center"
    },
    "5": { "fillColor": "#f2f2f2" },
    "6": { "fillColor": "#f2f2f2", "textColor": "" }
  },
  "formats": {},
  "borders": {
    "1": { "bottom": { "style": "thin", "color": "#CCCCCC" } },
    "2": { "top": { "style": "thin", "color": "#CCCCCC" } },
    "3": {
      "bottom": { "style": "thin", "color": "#FFFFFF" },
      "right": { "style": "thin", "color": "#FFFFFF" }
    },
    "4": {
      "top": { "style": "thin", "color": "#FFFFFF" },
      "bottom": { "style": "thin", "color": "#FFFFFF" },
      "right": { "style": "thin", "color": "#FFFFFF" }
    },
    "5": { "top": { "style": "thin", "color": "#FFFFFF" } },
    "6": {
      "bottom": { "style": "thin", "color": "#FFFFFF" },
      "left": { "style": "thin", "color": "#FFFFFF" },
      "right": { "style": "thin", "color": "#FFFFFF" }
    },
    "7": {
      "top": { "style": "thin", "color": "#FFFFFF" },
      "bottom": { "style": "thin", "color": "#FFFFFF" },
      "left": { "style": "thin", "color": "#FFFFFF" },
      "right": { "style": "thin", "color": "#FFFFFF" }
    },
    "8": {
      "bottom": { "style": "thin", "color": "#FFFFFF" },
      "left": { "style": "thin", "color": "#FFFFFF" }
    },
    "9": {
      "top": { "style": "thin", "color": "#FFFFFF" },
      "bottom": { "style": "thin", "color": "#FFFFFF" },
      "left": { "style": "thin", "color": "#FFFFFF" }
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
        "a6c3274b-e53c-4c6b-90a8-bc3e3d109f52": {
          "chain": "date",
          "type": "datetime",
          "offset": 0
        },
        "36814ad9-adac-4aba-a5ad-df595a306ef7": {
          "chain": "product_id",
          "type": "many2one"
        },
        "edaf4d0c-df4b-48bc-b61a-48a07e8afd91": {
          "chain": "categ_id",
          "type": "many2one"
        }
      },
      "context": { "group_by": [] },
      "domain": [["state", "not in", ["draft", "sent", "cancel"]]],
      "id": "1",
      "measures": [
        { "id": "product_uom_qty", "fieldName": "product_uom_qty" },
        { "id": "__count", "fieldName": "__count" },
        { "id": "price_subtotal", "fieldName": "price_subtotal" }
      ],
      "model": "sale.report",
      "name": "Sales Analysis by Product Variant",
      "sortedColumn": {
        "groupId": [[], []],
        "measure": "price_subtotal",
        "order": "desc"
      },
      "formulaId": "1",
      "columns": [],
      "rows": [{ "fieldName": "product_id" }]
    },
    "2": {
      "type": "ODOO",
      "fieldMatching": {
        "a6c3274b-e53c-4c6b-90a8-bc3e3d109f52": {
          "chain": "date",
          "type": "datetime",
          "offset": 0
        },
        "36814ad9-adac-4aba-a5ad-df595a306ef7": {
          "chain": "product_id",
          "type": "many2one"
        },
        "edaf4d0c-df4b-48bc-b61a-48a07e8afd91": {
          "chain": "categ_id",
          "type": "many2one"
        }
      },
      "context": { "group_by": [] },
      "domain": [["state", "not in", ["draft", "sent", "cancel"]]],
      "id": "2",
      "measures": [
        { "id": "product_uom_qty", "fieldName": "product_uom_qty" },
        { "id": "__count", "fieldName": "__count" },
        { "id": "price_subtotal", "fieldName": "price_subtotal" }
      ],
      "model": "sale.report",
      "name": "Sales Analysis by Product Category",
      "sortedColumn": {
        "groupId": [[], []],
        "measure": "price_subtotal",
        "order": "desc"
      },
      "formulaId": "2",
      "columns": [],
      "rows": [{ "fieldName": "categ_id" }]
    }
  },
  "pivotNextId": 3,
  "customTableStyles": {},
  "odooVersion": 12,
  "globalFilters": [
    {
      "id": "a6c3274b-e53c-4c6b-90a8-bc3e3d109f52",
      "type": "date",
      "label": "Period",
      "defaultValue": "last_month",
      "rangeType": "relative"
    },
    {
      "id": "36814ad9-adac-4aba-a5ad-df595a306ef7",
      "type": "relation",
      "label": "Product",
      "modelName": "product.product",
      "defaultValue": [],
      "defaultValueDisplayNames": [],
      "rangeType": "year"
    },
    {
      "id": "edaf4d0c-df4b-48bc-b61a-48a07e8afd91",
      "type": "relation",
      "label": "Category",
      "modelName": "product.category",
      "defaultValue": [],
      "defaultValueDisplayNames": [],
      "rangeType": "year"
    }
  ],
  "lists": {},
  "listNextId": 1,
  "chartOdooMenusReferences": {
    "c30175e7-604c-4fc7-8adb-03be67f0dc8f": "sale.sale_menu_root",
    "30212d4a-1f77-4cb8-8590-4eb34876e260": "sale.sale_menu_root",
    "e35856cf-9090-489b-b055-2d441380d954": "sale.sale_menu_root",
    "d0171069-d2cd-4c2c-a686-cd7515e93bb5": "sale.sale_menu_root",
    "5f383918-4073-4f19-9cc9-603216c953ad": "sale.menu_reporting_sales",
    "ec57f69b-f2b1-4dfc-990c-91ab61b526bf": "sale.menu_reporting_sales"
  }
}

```

## File: data\files\product_sample_dashboard.json

```json
{
    "version": 21,
    "sheets": [
        {
            "id": "sheet1",
            "name": "Dashboard",
            "colNumber": 7,
            "rowNumber": 60,
            "rows": {
                "6": {
                    "size": 40
                },
                "26": {
                    "size": 40
                },
                "46": {
                    "size": 40
                },
                "47": {
                    "size": 40
                },
                "48": {
                    "size": 29
                },
                "49": {
                    "size": 29
                },
                "50": {
                    "size": 29
                },
                "51": {
                    "size": 29
                },
                "52": {
                    "size": 29
                },
                "53": {
                    "size": 29
                },
                "54": {
                    "size": 29
                },
                "55": {
                    "size": 29
                },
                "56": {
                    "size": 29
                },
                "57": {
                    "size": 29
                }
            },
            "cols": {
                "0": {
                    "size": 275
                },
                "1": {
                    "size": 100
                },
                "2": {
                    "size": 100
                },
                "3": {
                    "size": 50
                },
                "4": {
                    "size": 275
                },
                "5": {
                    "size": 100
                },
                "6": {
                    "size": 100
                }
            },
            "merges": [],
            "cells": {
                "A7": {
                    "content": "[Best Sellers by Revenue](odoo://view/{\"viewType\":\"graph\",\"action\":{\"domain\":[[\"state\", \"not in\", [\"draft\", \"sent\", \"cancel\"]]],\"context\":{\"group_by\":[\"product_id\"],\"graph_measure\":\"price_subtotal\",\"graph_mode\":\"bar\",\"graph_groupbys\":[\"product_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Best Sellers by Revenue\"})"
                },
                "A27": {
                    "content": "[Best Sellers by Units Sold](odoo://view/{\"viewType\":\"graph\",\"action\":{\"domain\":[[\"state\", \"not in\", [\"draft\", \"sent\", \"cancel\"]]],\"context\":{\"group_by\":[\"product_id\"],\"graph_measure\":\"__count\",\"graph_mode\":\"bar\",\"graph_groupbys\":[\"product_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
                },
                "A47": {
                    "content": "[Best Selling Products](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"state\", \"not in\", [\"draft\", \"sent\", \"cancel\"]]],\"context\":{\"group_by\":[\"product_id\"],\"pivot_measures\":[\"__count\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"product_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
                },
                "A48": {
                    "content": "=_t(\"Product\")"
                },
                "B48": {
                    "content": "=_t(\"Units\")"
                },
                "C48": {
                    "content": "=_t(\"Revenue\")"
                },
                "E47": {
                    "content": "[Best Selling Categories](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"state\", \"not in\", [\"draft\", \"sent\", \"cancel\"]]],\"context\":{\"group_by\":[\"categ_id\"],\"pivot_measures\":[\"__count\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"categ_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
                },
                "E48": {
                    "content": "=_t(\"Category\")"
                },
                "F48": {
                    "content": "=_t(\"Units\")"
                },
                "G48": {
                    "content": "=_t(\"Revenue\")"
                }
            },
            "styles": {
                "A7": 1,
                "A27": 1,
                "A47": 1,
                "E47": 1,
                "A48": 2,
                "E48": 2,
                "B48:C48": 3,
                "F48:G48": 3
            },
            "formats": {},
            "borders": {
                "A47:C47": 1,
                "A7:G7": 1,
                "A27:G27": 1,
                "E47:G47": 1,
                "A48:C48": 2,
                "A8:G8": 2,
                "A28:G28": 2,
                "E48:G48": 2,
                "A49": 3,
                "E49": 3,
                "A50:A52": 4,
                "A54:A58": 4,
                "E50:E52": 4,
                "E54:E58": 4,
                "A53:C53": 5,
                "E53:G53": 5,
                "A59:C59": 6,
                "E59:G59": 6,
                "B49": 7,
                "F49": 7,
                "B50:B52": 8,
                "B54:B58": 8,
                "F50:F52": 8,
                "F54:F58": 8,
                "C49": 9,
                "G49": 9,
                "C50:C52": 10,
                "C54:C58": 10,
                "G50:G52": 10,
                "G54:G58": 10
            },
            "conditionalFormats": [],
            "figures": [
                {
                    "id": "5f383918-4073-4f19-9cc9-603216c953ad",
                    "x": 0,
                    "y": 12,
                    "width": 450,
                    "height": 108,
                    "tag": "chart",
                    "data": {
                        "baselineColorDown": "#DC6965",
                        "baselineColorUp": "#00A04A",
                        "baselineMode": "text",
                        "title": {
                            "text": "Best Seller",
                            "color": "#434343",
                            "bold": true
                        },
                        "type": "scorecard",
                        "background": "#FEF2F2",
                        "baseline": "Data!C2",
                        "baselineDescr": "sold",
                        "keyValue": "Data!B2",
                        "humanize": false
                    }
                },
                {
                    "id": "ec57f69b-f2b1-4dfc-990c-91ab61b526bf",
                    "x": 459,
                    "y": 12,
                    "width": 450,
                    "height": 108,
                    "tag": "chart",
                    "data": {
                        "baselineColorDown": "#DC6965",
                        "baselineColorUp": "#00A04A",
                        "baselineMode": "text",
                        "title": {
                            "text": "Best Category",
                            "color": "#434343",
                            "bold": true
                        },
                        "type": "scorecard",
                        "background": "#FEF2F2",
                        "baseline": "Data!C3",
                        "baselineDescr": "sold",
                        "keyValue": "Data!B3",
                        "humanize": false
                    }
                },
                {
                    "id": "d710386b-9584-44aa-a5ec-5bbc9e4312d1",
                    "x": 0,
                    "y": 178.05078125,
                    "width": 1000,
                    "height": 438,
                    "tag": "chart",
                    "data": {
                        "type": "bar",
                        "dataSetsHaveTitle": false,
                        "dataSets": [
                            {
                                "dataRange": "Data!B7:B26",
                                "yAxisId": "y"
                            }
                        ],
                        "legendPosition": "none",
                        "labelRange": "Data!A7:A26",
                        "title": {},
                        "stacked": false,
                        "aggregated": false
                    }
                },
                {
                    "id": "66891187-fb5e-4190-b115-c15e37a0df97",
                    "x": 0,
                    "y": 655,
                    "width": 1000,
                    "height": 437,
                    "tag": "chart",
                    "data": {
                        "type": "bar",
                        "dataSetsHaveTitle": false,
                        "dataSets": [
                            {
                                "dataRange": "Data!B29:B48",
                                "yAxisId": "y"
                            }
                        ],
                        "legendPosition": "none",
                        "labelRange": "Data!A29:A48",
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
            "id": "a83a78f2-b124-4d6f-9726-125e62a32b8d",
            "name": "Data",
            "colNumber": 23,
            "rowNumber": 88,
            "rows": {},
            "cols": {},
            "merges": [],
            "cells": {
                "A1": {
                    "content": "=_t(\"KPI\")"
                },
                "A2": {
                    "content": "=_t(\"Best selling product\")"
                },
                "A3": {
                    "content": "=_t(\"Best selling category\")"
                },
                "A7": {
                    "content": "=_t(\"TitanForge Gaming Chair\")"
                },
                "A8": {
                    "content": "=_t(\"GlideSync Wireless Mouse\")"
                },
                "A9": {
                    "content": "=_t(\"PulseFit Smartband\")"
                },
                "A10": {
                    "content": "=_t(\"CrystalWave Smart Mirror\")"
                },
                "A11": {
                    "content": "=_t(\"NovaTech Power Bank\")"
                },
                "A12": {
                    "content": "=_t(\"UltraBeam Projector\")"
                },
                "A13": {
                    "content": "=_t(\"VeloCharge Electric Bike\")"
                },
                "A14": {
                    "content": "=_t(\"QuantumSound Earbuds\")"
                },
                "A15": {
                    "content": "=_t(\"BreezePure Air Filter\")"
                },
                "A16": {
                    "content": "=_t(\"FlexiDesk Standing Desk\")"
                },
                "A17": {
                    "content": "=_t(\"AeroTrack Fitness Watch\")"
                },
                "A18": {
                    "content": "=_t(\"AeroMax Travel Pillow\")"
                },
                "A19": {
                    "content": "=_t(\"PureSonic Bluetooth Speaker\")"
                },
                "A20": {
                    "content": "=_t(\"HydroLux Water Bottle\")"
                },
                "A21": {
                    "content": "=_t(\"OmniClean Robot Vacuum\")"
                },
                "A22": {
                    "content": "=_t(\"SolarSwift Charger\")"
                },
                "A23": {
                    "content": "=_t(\"HyperChill Mini Fridge\")"
                },
                "A24": {
                    "content": "=_t(\"FlexiGrip Yoga Mat\")"
                },
                "A25": {
                    "content": "=_t(\"SnapGrip Camera Mount\")"
                },
                "A26": {
                    "content": "=_t(\"EcoBlade Kitchen Knife\")"
                },
                "A29": {
                    "content": "=_t(\"GlideSync Wireless Mouse\")"
                },
                "A30": {
                    "content": "=_t(\"TitanForge Gaming Chair\")"
                },
                "A31": {
                    "content": "=_t(\"PulseFit Smartband\")"
                },
                "A32": {
                    "content": "=_t(\"CrystalWave Smart Mirror\")"
                },
                "A33": {
                    "content": "=_t(\"NovaTech Power Bank\")"
                },
                "A34": {
                    "content": "=_t(\"UltraBeam Projector\")"
                },
                "A35": {
                    "content": "=_t(\"VeloCharge Electric Bike\")"
                },
                "A36": {
                    "content": "=_t(\"QuantumSound Earbuds\")"
                },
                "A37": {
                    "content": "=_t(\"BreezePure Air Filter\")"
                },
                "A38": {
                    "content": "=_t(\"FlexiDesk Standing Desk\")"
                },
                "A39": {
                    "content": "=_t(\"AeroTrack Fitness Watch\")"
                },
                "A40": {
                    "content": "=_t(\"AeroMax Travel Pillow\")"
                },
                "A41": {
                    "content": "=_t(\"PureSonic Bluetooth Speaker\")"
                },
                "A42": {
                    "content": "=_t(\"HydroLux Water Bottle\")"
                },
                "A43": {
                    "content": "=_t(\"OmniClean Robot Vacuum\")"
                },
                "A44": {
                    "content": "=_t(\"SolarSwift Charger\")"
                },
                "A45": {
                    "content": "=_t(\"HyperChill Mini Fridge\")"
                },
                "A46": {
                    "content": "=_t(\"FlexiGrip Yoga Mat\")"
                },
                "A47": {
                    "content": "=_t(\"SnapGrip Camera Mount\")"
                },
                "A48": {
                    "content": "=_t(\"EcoBlade Kitchen Knife\")"
                },
                "B1": {
                    "content": "=_t(\"Name\")"
                },
                "B2": {
                    "content": "=_t(\"GlideSync Wireless Mouse\")"
                },
                "B3": {
                    "content": "=_t(\"TitanForge Gaming Chair\")"
                },
                "B7": {
                    "content": "150000"
                },
                "B8": {
                    "content": "145000"
                },
                "B9": {
                    "content": "140000"
                },
                "B10": {
                    "content": "138000"
                },
                "B11": {
                    "content": "125000"
                },
                "B12": {
                    "content": "125000"
                },
                "B13": {
                    "content": "118000"
                },
                "B14": {
                    "content": "110000"
                },
                "B15": {
                    "content": "95000"
                },
                "B16": {
                    "content": "98000"
                },
                "B17": {
                    "content": "85500"
                },
                "B18": {
                    "content": "85000"
                },
                "B19": {
                    "content": "74000"
                },
                "B20": {
                    "content": "65000"
                },
                "B21": {
                    "content": "60000"
                },
                "B22": {
                    "content": "45000"
                },
                "B23": {
                    "content": "30000"
                },
                "B24": {
                    "content": "25000"
                },
                "B25": {
                    "content": "15500"
                },
                "B26": {
                    "content": "7500"
                },
                "B29": {
                    "content": "500"
                },
                "B30": {
                    "content": "475"
                },
                "B31": {
                    "content": "460"
                },
                "B32": {
                    "content": "445"
                },
                "B33": {
                    "content": "420"
                },
                "B34": {
                    "content": "410"
                },
                "B35": {
                    "content": "390"
                },
                "B36": {
                    "content": "365"
                },
                "B37": {
                    "content": "350"
                },
                "B38": {
                    "content": "330"
                },
                "B39": {
                    "content": "315"
                },
                "B40": {
                    "content": "290"
                },
                "B41": {
                    "content": "270"
                },
                "B42": {
                    "content": "250"
                },
                "B43": {
                    "content": "225"
                },
                "B44": {
                    "content": "195"
                },
                "B45": {
                    "content": "150"
                },
                "B46": {
                    "content": "120"
                },
                "B47": {
                    "content": "95"
                },
                "B48": {
                    "content": "60"
                },
                "C1": {
                    "content": "=_t(\"Units\")"
                },
                "C2": {
                    "content": "500"
                },
                "C3": {
                    "content": "475"
                },
                "D1": {
                    "content": "=_t(\"Revenue\")"
                },
                "D2": {
                    "content": "110000"
                },
                "D3": {
                    "content": "311155"
                }
            },
            "styles": {},
            "formats": {
                "B7:B26": 1,
                "D2:D3": 1
            },
            "borders": {},
            "conditionalFormats": [],
            "figures": [
                {
                    "id": "7283c8f8-6ecc-4537-90c3-2a8fd8c3a431",
                    "x": 660,
                    "y": 376.5,
                    "width": 536,
                    "height": 335,
                    "tag": "chart",
                    "data": {
                        "type": "bar",
                        "dataSetsHaveTitle": false,
                        "dataSets": [
                            {
                                "dataRange": "B29:B48",
                                "yAxisId": "y"
                            }
                        ],
                        "legendPosition": "none",
                        "labelRange": "A29:A48",
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
        }
    ],
    "styles": {
        "1": {
            "textColor": "#01666b",
            "bold": true,
            "fontSize": 16
        },
        "2": {
            "textColor": "#434343",
            "fontSize": 11,
            "bold": true
        },
        "3": {
            "textColor": "#434343",
            "fontSize": 11,
            "bold": true,
            "align": "center"
        }
    },
    "formats": {
        "1": "[$$]#,##0"
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
                "style": "thin",
                "color": "#FFFFFF"
            },
            "right": {
                "style": "thin",
                "color": "#FFFFFF"
            }
        },
        "4": {
            "top": {
                "style": "thin",
                "color": "#FFFFFF"
            },
            "bottom": {
                "style": "thin",
                "color": "#FFFFFF"
            },
            "right": {
                "style": "thin",
                "color": "#FFFFFF"
            }
        },
        "5": {
            "top": {
                "style": "thin",
                "color": "#FFFFFF"
            },
            "bottom": {
                "style": "thin",
                "color": "#FFFFFF"
            }
        },
        "6": {
            "top": {
                "style": "thin",
                "color": "#FFFFFF"
            }
        },
        "7": {
            "bottom": {
                "style": "thin",
                "color": "#FFFFFF"
            },
            "left": {
                "style": "thin",
                "color": "#FFFFFF"
            },
            "right": {
                "style": "thin",
                "color": "#FFFFFF"
            }
        },
        "8": {
            "top": {
                "style": "thin",
                "color": "#FFFFFF"
            },
            "bottom": {
                "style": "thin",
                "color": "#FFFFFF"
            },
            "left": {
                "style": "thin",
                "color": "#FFFFFF"
            },
            "right": {
                "style": "thin",
                "color": "#FFFFFF"
            }
        },
        "9": {
            "bottom": {
                "style": "thin",
                "color": "#FFFFFF"
            },
            "left": {
                "style": "thin",
                "color": "#FFFFFF"
            }
        },
        "10": {
            "top": {
                "style": "thin",
                "color": "#FFFFFF"
            },
            "bottom": {
                "style": "thin",
                "color": "#FFFFFF"
            },
            "left": {
                "style": "thin",
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
    "pivotNextId": 3,
    "customTableStyles": {},
    "odooVersion": 12,
    "globalFilters": [],
    "lists": {},
    "listNextId": 1
}

```

## File: data\files\sales_dashboard.json

```json
{
  "version": 21,
  "sheets": [
    {
      "id": "sheet1",
      "name": "Dashboard",
      "colNumber": 7,
      "rowNumber": 96,
      "rows": {
        "5": { "size": 40 },
        "21": { "size": 40 },
        "22": { "size": 29 },
        "23": { "size": 29 },
        "24": { "size": 29 },
        "25": { "size": 29 },
        "26": { "size": 29 },
        "27": { "size": 29 },
        "28": { "size": 29 },
        "29": { "size": 29 },
        "30": { "size": 29 },
        "31": { "size": 29 },
        "32": { "size": 29 },
        "33": { "size": 23 },
        "34": { "size": 43 },
        "35": { "size": 35 },
        "36": { "size": 28 },
        "37": { "size": 28 },
        "38": { "size": 28 },
        "39": { "size": 28 },
        "40": { "size": 28 },
        "41": { "size": 28 },
        "42": { "size": 28 },
        "43": { "size": 28 },
        "44": { "size": 28 },
        "45": { "size": 28 },
        "47": { "size": 40 },
        "48": { "size": 40 },
        "49": { "size": 28 },
        "50": { "size": 28 },
        "51": { "size": 28 },
        "52": { "size": 28 },
        "53": { "size": 28 },
        "54": { "size": 28 },
        "55": { "size": 28 },
        "56": { "size": 28 },
        "57": { "size": 28 },
        "58": { "size": 28 },
        "60": { "size": 40 },
        "61": { "size": 40 },
        "62": { "size": 28 },
        "63": { "size": 28 },
        "64": { "size": 28 },
        "65": { "size": 28 },
        "66": { "size": 28 },
        "67": { "size": 28 },
        "68": { "size": 28 },
        "69": { "size": 28 },
        "70": { "size": 28 },
        "71": { "size": 28 },
        "73": { "size": 40 },
        "74": { "size": 40 },
        "75": { "size": 28 },
        "76": { "size": 28 },
        "77": { "size": 28 },
        "78": { "size": 28 },
        "79": { "size": 28 },
        "80": { "size": 28 },
        "81": { "size": 28 },
        "82": { "size": 28 },
        "83": { "size": 28 },
        "84": { "size": 28 },
        "85": { "size": 28 },
        "86": { "size": 40 },
        "87": { "size": 40 }
      },
      "cols": {
        "0": { "size": 349 },
        "1": { "size": 95 },
        "2": { "size": 80 },
        "3": { "size": 50 },
        "4": { "size": 323 },
        "5": { "size": 100 },
        "6": { "size": 100 }
      },
      "merges": [],
      "cells": {
        "A6": {
          "content": "[Monthly Sales](odoo://view/{\"viewType\":\"graph\",\"action\":{\"domain\":[[\"state\",\"not in\",[\"draft\",\"cancel\",\"sent\"]]],\"context\":{\"group_by\":[\"date:month\"],\"graph_measure\":\"price_subtotal\",\"graph_mode\":\"line\",\"graph_groupbys\":[\"date:month\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
        },
        "A22": {
          "content": "[Top Quotations](odoo://view/{\"viewType\":\"list\",\"action\":{\"domain\":[[\"state\",\"in\",[\"draft\",\"sent\"]]],\"context\":{\"group_by\":[]},\"modelName\":\"sale.order\",\"views\":[[false,\"list\"],[false,\"kanban\"],[false,\"form\"],[false,\"calendar\"],[false,\"pivot\"],[false,\"graph\"],[false,\"activity\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Quotations\"})"
        },
        "A23": { "content": "=_t(\"Customer\")" },
        "A24": { "content": "=ODOO.LIST(1,1,\"partner_id\")" },
        "A25": { "content": "=ODOO.LIST(1,2,\"partner_id\")" },
        "A26": { "content": "=ODOO.LIST(1,3,\"partner_id\")" },
        "A27": { "content": "=ODOO.LIST(1,4,\"partner_id\")" },
        "A28": { "content": "=ODOO.LIST(1,5,\"partner_id\")" },
        "A29": { "content": "=ODOO.LIST(1,6,\"partner_id\")" },
        "A30": { "content": "=ODOO.LIST(1,7,\"partner_id\")" },
        "A31": { "content": "=ODOO.LIST(1,8,\"partner_id\")" },
        "A32": { "content": "=ODOO.LIST(1,9,\"partner_id\")" },
        "A33": { "content": "=ODOO.LIST(1,10,\"partner_id\")" },
        "A35": {
          "content": "[Top Countries](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"country_id\",\"!=\",false],[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[\"country_id\"],\"pivot_measures\":[\"order_reference\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"country_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
        },
        "A36": { "content": "=_t(\"Country\")" },
        "A37": { "content": "=PIVOT.HEADER(5,\"#country_id\",1)" },
        "A38": { "content": "=PIVOT.HEADER(5,\"#country_id\",2)" },
        "A39": { "content": "=PIVOT.HEADER(5,\"#country_id\",3)" },
        "A40": { "content": "=PIVOT.HEADER(5,\"#country_id\",4)" },
        "A41": { "content": "=PIVOT.HEADER(5,\"#country_id\",5)" },
        "A42": { "content": "=PIVOT.HEADER(5,\"#country_id\",6)" },
        "A43": { "content": "=PIVOT.HEADER(5,\"#country_id\",7)" },
        "A44": { "content": "=PIVOT.HEADER(5,\"#country_id\",8)" },
        "A45": { "content": "=PIVOT.HEADER(5,\"#country_id\",9)" },
        "A46": { "content": "=PIVOT.HEADER(5,\"#country_id\",10)" },
        "A48": {
          "content": "[Top Customers](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"partner_id\",\"!=\",false],[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[\"partner_id\"],\"pivot_measures\":[\"order_reference\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"partner_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
        },
        "A49": { "content": "=_t(\"Customer\")" },
        "A50": { "content": "=PIVOT.HEADER(4,\"#partner_id\",1)" },
        "A51": { "content": "=PIVOT.HEADER(4,\"#partner_id\",2)" },
        "A52": { "content": "=PIVOT.HEADER(4,\"#partner_id\",3)" },
        "A53": { "content": "=PIVOT.HEADER(4,\"#partner_id\",4)" },
        "A54": { "content": "=PIVOT.HEADER(4,\"#partner_id\",5)" },
        "A55": { "content": "=PIVOT.HEADER(4,\"#partner_id\",6)" },
        "A56": { "content": "=PIVOT.HEADER(4,\"#partner_id\",7)" },
        "A57": { "content": "=PIVOT.HEADER(4,\"#partner_id\",8)" },
        "A58": { "content": "=PIVOT.HEADER(4,\"#partner_id\",9)" },
        "A59": { "content": "=PIVOT.HEADER(4,\"#partner_id\",10)" },
        "A61": {
          "content": "[Top Sales Teams](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"team_id\",\"!=\",false],[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[\"team_id\"],\"pivot_measures\":[\"order_reference\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"team_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
        },
        "A62": { "content": "=_t(\"Sales Team\")" },
        "A63": { "content": "=PIVOT.HEADER(7,\"#team_id\",1)" },
        "A64": { "content": "=PIVOT.HEADER(7,\"#team_id\",2)" },
        "A65": { "content": "=PIVOT.HEADER(7,\"#team_id\",3)" },
        "A66": { "content": "=PIVOT.HEADER(7,\"#team_id\",4)" },
        "A67": { "content": "=PIVOT.HEADER(7,\"#team_id\",5)" },
        "A68": { "content": "=PIVOT.HEADER(7,\"#team_id\",6)" },
        "A69": { "content": "=PIVOT.HEADER(7,\"#team_id\",7)" },
        "A70": { "content": "=PIVOT.HEADER(7,\"#team_id\",8)" },
        "A71": { "content": "=PIVOT.HEADER(7,\"#team_id\",9)" },
        "A72": { "content": "=PIVOT.HEADER(7,\"#team_id\",10)" },
        "A74": {
          "content": "[Top Sources](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"source_id\",\"!=\",false],[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[\"source_id\"],\"pivot_measures\":[\"order_reference\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"source_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
        },
        "A75": { "content": "=_t(\"Source\")" },
        "A76": { "content": "=PIVOT.HEADER(9,\"#source_id\",1)" },
        "A77": { "content": "=PIVOT.HEADER(9,\"#source_id\",2)" },
        "A78": { "content": "=PIVOT.HEADER(9,\"#source_id\",3)" },
        "A79": { "content": "=PIVOT.HEADER(9,\"#source_id\",4)" },
        "A80": { "content": "=PIVOT.HEADER(9,\"#source_id\",5)" },
        "A81": { "content": "=PIVOT.HEADER(9,\"#source_id\",6)" },
        "A82": { "content": "=PIVOT.HEADER(9,\"#source_id\",7)" },
        "A83": { "content": "=PIVOT.HEADER(9,\"#source_id\",8)" },
        "A84": { "content": "=PIVOT.HEADER(9,\"#source_id\",9)" },
        "A85": { "content": "=PIVOT.HEADER(9,\"#source_id\",10)" },
        "B23": { "content": "=_t(\"Salesperson\")" },
        "B24": { "content": "=ODOO.LIST(1,1,\"user_id\")" },
        "B25": { "content": "=ODOO.LIST(1,2,\"user_id\")" },
        "B26": { "content": "=ODOO.LIST(1,3,\"user_id\")" },
        "B27": { "content": "=ODOO.LIST(1,4,\"user_id\")" },
        "B28": { "content": "=ODOO.LIST(1,5,\"user_id\")" },
        "B29": { "content": "=ODOO.LIST(1,6,\"user_id\")" },
        "B30": { "content": "=ODOO.LIST(1,7,\"user_id\")" },
        "B31": { "content": "=ODOO.LIST(1,8,\"user_id\")" },
        "B32": { "content": "=ODOO.LIST(1,9,\"user_id\")" },
        "B33": { "content": "=ODOO.LIST(1,10,\"user_id\")" },
        "B36": { "content": "=_t(\"Orders\")" },
        "B37": {
          "content": "=PIVOT.VALUE(5,\"order_reference\",\"#country_id\",1)"
        },
        "B38": {
          "content": "=PIVOT.VALUE(5,\"order_reference\",\"#country_id\",2)"
        },
        "B39": {
          "content": "=PIVOT.VALUE(5,\"order_reference\",\"#country_id\",3)"
        },
        "B40": {
          "content": "=PIVOT.VALUE(5,\"order_reference\",\"#country_id\",4)"
        },
        "B41": {
          "content": "=PIVOT.VALUE(5,\"order_reference\",\"#country_id\",5)"
        },
        "B42": {
          "content": "=PIVOT.VALUE(5,\"order_reference\",\"#country_id\",6)"
        },
        "B43": {
          "content": "=PIVOT.VALUE(5,\"order_reference\",\"#country_id\",7)"
        },
        "B44": {
          "content": "=PIVOT.VALUE(5,\"order_reference\",\"#country_id\",8)"
        },
        "B45": {
          "content": "=PIVOT.VALUE(5,\"order_reference\",\"#country_id\",9)"
        },
        "B46": {
          "content": "=PIVOT.VALUE(5,\"order_reference\",\"#country_id\",10)"
        },
        "B49": { "content": "=_t(\"Orders\")" },
        "B50": {
          "content": "=PIVOT.VALUE(4,\"order_reference\",\"#partner_id\",1)"
        },
        "B51": {
          "content": "=PIVOT.VALUE(4,\"order_reference\",\"#partner_id\",2)"
        },
        "B52": {
          "content": "=PIVOT.VALUE(4,\"order_reference\",\"#partner_id\",3)"
        },
        "B53": {
          "content": "=PIVOT.VALUE(4,\"order_reference\",\"#partner_id\",4)"
        },
        "B54": {
          "content": "=PIVOT.VALUE(4,\"order_reference\",\"#partner_id\",5)"
        },
        "B55": {
          "content": "=PIVOT.VALUE(4,\"order_reference\",\"#partner_id\",6)"
        },
        "B56": {
          "content": "=PIVOT.VALUE(4,\"order_reference\",\"#partner_id\",7)"
        },
        "B57": {
          "content": "=PIVOT.VALUE(4,\"order_reference\",\"#partner_id\",8)"
        },
        "B58": {
          "content": "=PIVOT.VALUE(4,\"order_reference\",\"#partner_id\",9)"
        },
        "B59": {
          "content": "=PIVOT.VALUE(4,\"order_reference\",\"#partner_id\",10)"
        },
        "B62": { "content": "=_t(\"Orders\")" },
        "B63": {
          "content": "=PIVOT.VALUE(7,\"order_reference\",\"#team_id\",1)"
        },
        "B64": {
          "content": "=PIVOT.VALUE(7,\"order_reference\",\"#team_id\",2)"
        },
        "B65": {
          "content": "=PIVOT.VALUE(7,\"order_reference\",\"#team_id\",3)"
        },
        "B66": {
          "content": "=PIVOT.VALUE(7,\"order_reference\",\"#team_id\",4)"
        },
        "B67": {
          "content": "=PIVOT.VALUE(7,\"order_reference\",\"#team_id\",5)"
        },
        "B68": {
          "content": "=PIVOT.VALUE(7,\"order_reference\",\"#team_id\",6)"
        },
        "B69": {
          "content": "=PIVOT.VALUE(7,\"order_reference\",\"#team_id\",7)"
        },
        "B70": {
          "content": "=PIVOT.VALUE(7,\"order_reference\",\"#team_id\",8)"
        },
        "B71": {
          "content": "=PIVOT.VALUE(7,\"order_reference\",\"#team_id\",9)"
        },
        "B72": {
          "content": "=PIVOT.VALUE(7,\"order_reference\",\"#team_id\",10)"
        },
        "B75": { "content": "=_t(\"Orders\")" },
        "B76": {
          "content": "=PIVOT.VALUE(9,\"order_reference\",\"#source_id\",1)"
        },
        "B77": {
          "content": "=PIVOT.VALUE(9,\"order_reference\",\"#source_id\",2)"
        },
        "B78": {
          "content": "=PIVOT.VALUE(9,\"order_reference\",\"#source_id\",3)"
        },
        "B79": {
          "content": "=PIVOT.VALUE(9,\"order_reference\",\"#source_id\",4)"
        },
        "B80": {
          "content": "=PIVOT.VALUE(9,\"order_reference\",\"#source_id\",5)"
        },
        "B81": {
          "content": "=PIVOT.VALUE(9,\"order_reference\",\"#source_id\",6)"
        },
        "B82": {
          "content": "=PIVOT.VALUE(9,\"order_reference\",\"#source_id\",7)"
        },
        "B83": {
          "content": "=PIVOT.VALUE(9,\"order_reference\",\"#source_id\",8)"
        },
        "B84": {
          "content": "=PIVOT.VALUE(9,\"order_reference\",\"#source_id\",9)"
        },
        "B85": {
          "content": "=PIVOT.VALUE(9,\"order_reference\",\"#source_id\",10)"
        },
        "C23": { "content": "=_t(\"Revenue\")" },
        "C24": { "content": "=ODOO.LIST(1,1,\"amount_untaxed\")" },
        "C25": { "content": "=ODOO.LIST(1,2,\"amount_untaxed\")" },
        "C26": { "content": "=ODOO.LIST(1,3,\"amount_untaxed\")" },
        "C27": { "content": "=ODOO.LIST(1,4,\"amount_untaxed\")" },
        "C28": { "content": "=ODOO.LIST(1,5,\"amount_untaxed\")" },
        "C29": { "content": "=ODOO.LIST(1,6,\"amount_untaxed\")" },
        "C30": { "content": "=ODOO.LIST(1,7,\"amount_untaxed\")" },
        "C31": { "content": "=ODOO.LIST(1,8,\"amount_untaxed\")" },
        "C32": { "content": "=ODOO.LIST(1,9,\"amount_untaxed\")" },
        "C33": { "content": "=ODOO.LIST(1,10,\"amount_untaxed\")" },
        "C36": { "content": "=_t(\"Revenue\")" },
        "C37": {
          "content": "=PIVOT.VALUE(5,\"price_subtotal\",\"#country_id\",1)"
        },
        "C38": {
          "content": "=PIVOT.VALUE(5,\"price_subtotal\",\"#country_id\",2)"
        },
        "C39": {
          "content": "=PIVOT.VALUE(5,\"price_subtotal\",\"#country_id\",3)"
        },
        "C40": {
          "content": "=PIVOT.VALUE(5,\"price_subtotal\",\"#country_id\",4)"
        },
        "C41": {
          "content": "=PIVOT.VALUE(5,\"price_subtotal\",\"#country_id\",5)"
        },
        "C42": {
          "content": "=PIVOT.VALUE(5,\"price_subtotal\",\"#country_id\",6)"
        },
        "C43": {
          "content": "=PIVOT.VALUE(5,\"price_subtotal\",\"#country_id\",7)"
        },
        "C44": {
          "content": "=PIVOT.VALUE(5,\"price_subtotal\",\"#country_id\",8)"
        },
        "C45": {
          "content": "=PIVOT.VALUE(5,\"price_subtotal\",\"#country_id\",9)"
        },
        "C46": {
          "content": "=PIVOT.VALUE(5,\"price_subtotal\",\"#country_id\",10)"
        },
        "C49": { "content": "=_t(\"Revenue\")" },
        "C50": {
          "content": "=PIVOT.VALUE(4,\"price_subtotal\",\"#partner_id\",1)"
        },
        "C51": {
          "content": "=PIVOT.VALUE(4,\"price_subtotal\",\"#partner_id\",2)"
        },
        "C52": {
          "content": "=PIVOT.VALUE(4,\"price_subtotal\",\"#partner_id\",3)"
        },
        "C53": {
          "content": "=PIVOT.VALUE(4,\"price_subtotal\",\"#partner_id\",4)"
        },
        "C54": {
          "content": "=PIVOT.VALUE(4,\"price_subtotal\",\"#partner_id\",5)"
        },
        "C55": {
          "content": "=PIVOT.VALUE(4,\"price_subtotal\",\"#partner_id\",6)"
        },
        "C56": {
          "content": "=PIVOT.VALUE(4,\"price_subtotal\",\"#partner_id\",7)"
        },
        "C57": {
          "content": "=PIVOT.VALUE(4,\"price_subtotal\",\"#partner_id\",8)"
        },
        "C58": {
          "content": "=PIVOT.VALUE(4,\"price_subtotal\",\"#partner_id\",9)"
        },
        "C59": {
          "content": "=PIVOT.VALUE(4,\"price_subtotal\",\"#partner_id\",10)"
        },
        "C62": { "content": "=_t(\"Revenue\")" },
        "C63": {
          "content": "=PIVOT.VALUE(7,\"price_subtotal\",\"#team_id\",1)"
        },
        "C64": {
          "content": "=PIVOT.VALUE(7,\"price_subtotal\",\"#team_id\",2)"
        },
        "C65": {
          "content": "=PIVOT.VALUE(7,\"price_subtotal\",\"#team_id\",3)"
        },
        "C66": {
          "content": "=PIVOT.VALUE(7,\"price_subtotal\",\"#team_id\",4)"
        },
        "C67": {
          "content": "=PIVOT.VALUE(7,\"price_subtotal\",\"#team_id\",5)"
        },
        "C68": {
          "content": "=PIVOT.VALUE(7,\"price_subtotal\",\"#team_id\",6)"
        },
        "C69": {
          "content": "=PIVOT.VALUE(7,\"price_subtotal\",\"#team_id\",7)"
        },
        "C70": {
          "content": "=PIVOT.VALUE(7,\"price_subtotal\",\"#team_id\",8)"
        },
        "C71": {
          "content": "=PIVOT.VALUE(7,\"price_subtotal\",\"#team_id\",9)"
        },
        "C72": {
          "content": "=PIVOT.VALUE(7,\"price_subtotal\",\"#team_id\",10)"
        },
        "C75": { "content": "=_t(\"Revenue\")" },
        "C76": {
          "content": "=PIVOT.VALUE(9,\"price_subtotal\",\"#source_id\",1)"
        },
        "C77": {
          "content": "=PIVOT.VALUE(9,\"price_subtotal\",\"#source_id\",2)"
        },
        "C78": {
          "content": "=PIVOT.VALUE(9,\"price_subtotal\",\"#source_id\",3)"
        },
        "C79": {
          "content": "=PIVOT.VALUE(9,\"price_subtotal\",\"#source_id\",4)"
        },
        "C80": {
          "content": "=PIVOT.VALUE(9,\"price_subtotal\",\"#source_id\",5)"
        },
        "C81": {
          "content": "=PIVOT.VALUE(9,\"price_subtotal\",\"#source_id\",6)"
        },
        "C82": {
          "content": "=PIVOT.VALUE(9,\"price_subtotal\",\"#source_id\",7)"
        },
        "C83": {
          "content": "=PIVOT.VALUE(9,\"price_subtotal\",\"#source_id\",8)"
        },
        "C84": {
          "content": "=PIVOT.VALUE(9,\"price_subtotal\",\"#source_id\",9)"
        },
        "C85": {
          "content": "=PIVOT.VALUE(9,\"price_subtotal\",\"#source_id\",10)"
        },
        "E22": {
          "content": "[Top Sales Orders](odoo://view/{\"viewType\":\"list\",\"action\":{\"domain\":[[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[]},\"modelName\":\"sale.order\",\"views\":[[false,\"list\"],[false,\"kanban\"],[false,\"form\"],[false,\"calendar\"],[false,\"pivot\"],[false,\"graph\"],[false,\"activity\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Orders\"})"
        },
        "E23": { "content": "=_t(\"Customer\")" },
        "E24": { "content": "=ODOO.LIST(2,1,\"partner_id\")" },
        "E25": { "content": "=ODOO.LIST(2,2,\"partner_id\")" },
        "E26": { "content": "=ODOO.LIST(2,3,\"partner_id\")" },
        "E27": { "content": "=ODOO.LIST(2,4,\"partner_id\")" },
        "E28": { "content": "=ODOO.LIST(2,5,\"partner_id\")" },
        "E29": { "content": "=ODOO.LIST(2,6,\"partner_id\")" },
        "E30": { "content": "=ODOO.LIST(2,7,\"partner_id\")" },
        "E31": { "content": "=ODOO.LIST(2,8,\"partner_id\")" },
        "E32": { "content": "=ODOO.LIST(2,9,\"partner_id\")" },
        "E33": { "content": "=ODOO.LIST(2,10,\"partner_id\")" },
        "E35": {
          "content": "[Top Products](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"product_tmpl_id\",\"!=\",false],[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[\"product_id\"],\"pivot_measures\":[\"order_reference\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"product_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
        },
        "E36": { "content": "=_t(\"Product\")" },
        "E37": { "content": "=PIVOT.HEADER(6,\"#product_id\",1)" },
        "E38": { "content": "=PIVOT.HEADER(6,\"#product_id\",2)" },
        "E39": { "content": "=PIVOT.HEADER(6,\"#product_id\",3)" },
        "E40": { "content": "=PIVOT.HEADER(6,\"#product_id\",4)" },
        "E41": { "content": "=PIVOT.HEADER(6,\"#product_id\",5)" },
        "E42": { "content": "=PIVOT.HEADER(6,\"#product_id\",6)" },
        "E43": { "content": "=PIVOT.HEADER(6,\"#product_id\",7)" },
        "E44": { "content": "=PIVOT.HEADER(6,\"#product_id\",8)" },
        "E45": { "content": "=PIVOT.HEADER(6,\"#product_id\",9)" },
        "E46": { "content": "=PIVOT.HEADER(6,\"#product_id\",10)" },
        "E48": {
          "content": "[Top Categories](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"categ_id\",\"!=\",false],[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[\"categ_id\"],\"pivot_measures\":[\"order_reference\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"categ_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
        },
        "E49": { "content": "=_t(\"Category\")" },
        "E50": { "content": "=PIVOT.HEADER(3,\"#categ_id\",1)" },
        "E51": { "content": "=PIVOT.HEADER(3,\"#categ_id\",2)" },
        "E52": { "content": "=PIVOT.HEADER(3,\"#categ_id\",3)" },
        "E53": { "content": "=PIVOT.HEADER(3,\"#categ_id\",4)" },
        "E54": { "content": "=PIVOT.HEADER(3,\"#categ_id\",5)" },
        "E55": { "content": "=PIVOT.HEADER(3,\"#categ_id\",6)" },
        "E56": { "content": "=PIVOT.HEADER(3,\"#categ_id\",7)" },
        "E57": { "content": "=PIVOT.HEADER(3,\"#categ_id\",8)" },
        "E58": { "content": "=PIVOT.HEADER(3,\"#categ_id\",9)" },
        "E59": { "content": "=PIVOT.HEADER(3,\"#categ_id\",10)" },
        "E61": {
          "content": "[Top Salespeople](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"user_id\",\"!=\",false],[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[\"user_id\"],\"pivot_measures\":[\"order_reference\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"user_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
        },
        "E62": { "content": "=_t(\"Salesperson\")" },
        "E63": { "content": "=PIVOT.HEADER(8,\"#user_id\",1)" },
        "E64": { "content": "=PIVOT.HEADER(8,\"#user_id\",2)" },
        "E65": { "content": "=PIVOT.HEADER(8,\"#user_id\",3)" },
        "E66": { "content": "=PIVOT.HEADER(8,\"#user_id\",4)" },
        "E67": { "content": "=PIVOT.HEADER(8,\"#user_id\",5)" },
        "E68": { "content": "=PIVOT.HEADER(8,\"#user_id\",6)" },
        "E69": { "content": "=PIVOT.HEADER(8,\"#user_id\",7)" },
        "E70": { "content": "=PIVOT.HEADER(8,\"#user_id\",8)" },
        "E71": { "content": "=PIVOT.HEADER(8,\"#user_id\",9)" },
        "E72": { "content": "=PIVOT.HEADER(8,\"#user_id\",10)" },
        "E74": {
          "content": "[Top Mediums](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"medium_id\",\"!=\",false],[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[\"medium_id\"],\"pivot_measures\":[\"order_reference\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"medium_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
        },
        "E75": { "content": "=_t(\"Medium\")" },
        "E76": { "content": "=PIVOT.HEADER(10,\"#medium_id\",1)" },
        "E77": { "content": "=PIVOT.HEADER(10,\"#medium_id\",2)" },
        "E78": { "content": "=PIVOT.HEADER(10,\"#medium_id\",3)" },
        "E79": { "content": "=PIVOT.HEADER(10,\"#medium_id\",4)" },
        "E80": { "content": "=PIVOT.HEADER(10,\"#medium_id\",5)" },
        "E81": { "content": "=PIVOT.HEADER(10,\"#medium_id\",6)" },
        "E82": { "content": "=PIVOT.HEADER(10,\"#medium_id\",7)" },
        "E83": { "content": "=PIVOT.HEADER(10,\"#medium_id\",8)" },
        "E84": { "content": "=PIVOT.HEADER(10,\"#medium_id\",9)" },
        "E85": { "content": "=PIVOT.HEADER(10,\"#medium_id\",10)" },
        "F23": { "content": "=_t(\"Salesperson\")" },
        "F24": { "content": "=ODOO.LIST(2,1,\"user_id\")" },
        "F25": { "content": "=ODOO.LIST(2,2,\"user_id\")" },
        "F26": { "content": "=ODOO.LIST(2,3,\"user_id\")" },
        "F27": { "content": "=ODOO.LIST(2,4,\"user_id\")" },
        "F28": { "content": "=ODOO.LIST(2,5,\"user_id\")" },
        "F29": { "content": "=ODOO.LIST(2,6,\"user_id\")" },
        "F30": { "content": "=ODOO.LIST(2,7,\"user_id\")" },
        "F31": { "content": "=ODOO.LIST(2,8,\"user_id\")" },
        "F32": { "content": "=ODOO.LIST(2,9,\"user_id\")" },
        "F33": { "content": "=ODOO.LIST(2,10,\"user_id\")" },
        "F36": { "content": "=_t(\"Orders\")" },
        "F37": {
          "content": "=PIVOT.VALUE(6,\"order_reference\",\"#product_id\",1)"
        },
        "F38": {
          "content": "=PIVOT.VALUE(6,\"order_reference\",\"#product_id\",2)"
        },
        "F39": {
          "content": "=PIVOT.VALUE(6,\"order_reference\",\"#product_id\",3)"
        },
        "F40": {
          "content": "=PIVOT.VALUE(6,\"order_reference\",\"#product_id\",4)"
        },
        "F41": {
          "content": "=PIVOT.VALUE(6,\"order_reference\",\"#product_id\",5)"
        },
        "F42": {
          "content": "=PIVOT.VALUE(6,\"order_reference\",\"#product_id\",6)"
        },
        "F43": {
          "content": "=PIVOT.VALUE(6,\"order_reference\",\"#product_id\",7)"
        },
        "F44": {
          "content": "=PIVOT.VALUE(6,\"order_reference\",\"#product_id\",8)"
        },
        "F45": {
          "content": "=PIVOT.VALUE(6,\"order_reference\",\"#product_id\",9)"
        },
        "F46": {
          "content": "=PIVOT.VALUE(6,\"order_reference\",\"#product_id\",10)"
        },
        "F49": { "content": "=_t(\"Orders\")" },
        "F50": {
          "content": "=PIVOT.VALUE(3,\"order_reference\",\"#categ_id\",1)"
        },
        "F51": {
          "content": "=PIVOT.VALUE(3,\"order_reference\",\"#categ_id\",2)"
        },
        "F52": {
          "content": "=PIVOT.VALUE(3,\"order_reference\",\"#categ_id\",3)"
        },
        "F53": {
          "content": "=PIVOT.VALUE(3,\"order_reference\",\"#categ_id\",4)"
        },
        "F54": {
          "content": "=PIVOT.VALUE(3,\"order_reference\",\"#categ_id\",5)"
        },
        "F55": {
          "content": "=PIVOT.VALUE(3,\"order_reference\",\"#categ_id\",6)"
        },
        "F56": {
          "content": "=PIVOT.VALUE(3,\"order_reference\",\"#categ_id\",7)"
        },
        "F57": {
          "content": "=PIVOT.VALUE(3,\"order_reference\",\"#categ_id\",8)"
        },
        "F58": {
          "content": "=PIVOT.VALUE(3,\"order_reference\",\"#categ_id\",9)"
        },
        "F59": {
          "content": "=PIVOT.VALUE(3,\"order_reference\",\"#categ_id\",10)"
        },
        "F62": { "content": "=_t(\"Orders\")" },
        "F63": {
          "content": "=PIVOT.VALUE(8,\"order_reference\",\"#user_id\",1)"
        },
        "F64": {
          "content": "=PIVOT.VALUE(8,\"order_reference\",\"#user_id\",2)"
        },
        "F65": {
          "content": "=PIVOT.VALUE(8,\"order_reference\",\"#user_id\",3)"
        },
        "F66": {
          "content": "=PIVOT.VALUE(8,\"order_reference\",\"#user_id\",4)"
        },
        "F67": {
          "content": "=PIVOT.VALUE(8,\"order_reference\",\"#user_id\",5)"
        },
        "F68": {
          "content": "=PIVOT.VALUE(8,\"order_reference\",\"#user_id\",6)"
        },
        "F69": {
          "content": "=PIVOT.VALUE(8,\"order_reference\",\"#user_id\",7)"
        },
        "F70": {
          "content": "=PIVOT.VALUE(8,\"order_reference\",\"#user_id\",8)"
        },
        "F71": {
          "content": "=PIVOT.VALUE(8,\"order_reference\",\"#user_id\",9)"
        },
        "F72": {
          "content": "=PIVOT.VALUE(8,\"order_reference\",\"#user_id\",10)"
        },
        "F75": { "content": "=_t(\"Orders\")" },
        "F76": {
          "content": "=PIVOT.VALUE(10,\"order_reference\",\"#medium_id\",1)"
        },
        "F77": {
          "content": "=PIVOT.VALUE(10,\"order_reference\",\"#medium_id\",2)"
        },
        "F78": {
          "content": "=PIVOT.VALUE(10,\"order_reference\",\"#medium_id\",3)"
        },
        "F79": {
          "content": "=PIVOT.VALUE(10,\"order_reference\",\"#medium_id\",4)"
        },
        "F80": {
          "content": "=PIVOT.VALUE(10,\"order_reference\",\"#medium_id\",5)"
        },
        "F81": {
          "content": "=PIVOT.VALUE(10,\"order_reference\",\"#medium_id\",6)"
        },
        "F82": {
          "content": "=PIVOT.VALUE(10,\"order_reference\",\"#medium_id\",7)"
        },
        "F83": {
          "content": "=PIVOT.VALUE(10,\"order_reference\",\"#medium_id\",8)"
        },
        "F84": {
          "content": "=PIVOT.VALUE(10,\"order_reference\",\"#medium_id\",9)"
        },
        "F85": {
          "content": "=PIVOT.VALUE(10,\"order_reference\",\"#medium_id\",10)"
        },
        "G23": { "content": "=_t(\"Revenue\")" },
        "G24": { "content": "=ODOO.LIST(2,1,\"amount_untaxed\")" },
        "G25": { "content": "=ODOO.LIST(2,2,\"amount_untaxed\")" },
        "G26": { "content": "=ODOO.LIST(2,3,\"amount_untaxed\")" },
        "G27": { "content": "=ODOO.LIST(2,4,\"amount_untaxed\")" },
        "G28": { "content": "=ODOO.LIST(2,5,\"amount_untaxed\")" },
        "G29": { "content": "=ODOO.LIST(2,6,\"amount_untaxed\")" },
        "G30": { "content": "=ODOO.LIST(2,7,\"amount_untaxed\")" },
        "G31": { "content": "=ODOO.LIST(2,8,\"amount_untaxed\")" },
        "G32": { "content": "=ODOO.LIST(2,9,\"amount_untaxed\")" },
        "G33": { "content": "=ODOO.LIST(2,10,\"amount_untaxed\")" },
        "G36": { "content": "=_t(\"Revenue\")" },
        "G37": {
          "content": "=PIVOT.VALUE(6,\"price_subtotal\",\"#product_id\",1)"
        },
        "G38": {
          "content": "=PIVOT.VALUE(6,\"price_subtotal\",\"#product_id\",2)"
        },
        "G39": {
          "content": "=PIVOT.VALUE(6,\"price_subtotal\",\"#product_id\",3)"
        },
        "G40": {
          "content": "=PIVOT.VALUE(6,\"price_subtotal\",\"#product_id\",4)"
        },
        "G41": {
          "content": "=PIVOT.VALUE(6,\"price_subtotal\",\"#product_id\",5)"
        },
        "G42": {
          "content": "=PIVOT.VALUE(6,\"price_subtotal\",\"#product_id\",6)"
        },
        "G43": {
          "content": "=PIVOT.VALUE(6,\"price_subtotal\",\"#product_id\",7)"
        },
        "G44": {
          "content": "=PIVOT.VALUE(6,\"price_subtotal\",\"#product_id\",8)"
        },
        "G45": {
          "content": "=PIVOT.VALUE(6,\"price_subtotal\",\"#product_id\",9)"
        },
        "G46": {
          "content": "=PIVOT.VALUE(6,\"price_subtotal\",\"#product_id\",10)"
        },
        "G49": { "content": "=_t(\"Revenue\")" },
        "G50": {
          "content": "=PIVOT.VALUE(3,\"price_subtotal\",\"#categ_id\",1)"
        },
        "G51": {
          "content": "=PIVOT.VALUE(3,\"price_subtotal\",\"#categ_id\",2)"
        },
        "G52": {
          "content": "=PIVOT.VALUE(3,\"price_subtotal\",\"#categ_id\",3)"
        },
        "G53": {
          "content": "=PIVOT.VALUE(3,\"price_subtotal\",\"#categ_id\",4)"
        },
        "G54": {
          "content": "=PIVOT.VALUE(3,\"price_subtotal\",\"#categ_id\",5)"
        },
        "G55": {
          "content": "=PIVOT.VALUE(3,\"price_subtotal\",\"#categ_id\",6)"
        },
        "G56": {
          "content": "=PIVOT.VALUE(3,\"price_subtotal\",\"#categ_id\",7)"
        },
        "G57": {
          "content": "=PIVOT.VALUE(3,\"price_subtotal\",\"#categ_id\",8)"
        },
        "G58": {
          "content": "=PIVOT.VALUE(3,\"price_subtotal\",\"#categ_id\",9)"
        },
        "G59": {
          "content": "=PIVOT.VALUE(3,\"price_subtotal\",\"#categ_id\",10)"
        },
        "G62": { "content": "=_t(\"Revenue\")" },
        "G63": {
          "content": "=PIVOT.VALUE(8,\"price_subtotal\",\"#user_id\",1)"
        },
        "G64": {
          "content": "=PIVOT.VALUE(8,\"price_subtotal\",\"#user_id\",2)"
        },
        "G65": {
          "content": "=PIVOT.VALUE(8,\"price_subtotal\",\"#user_id\",3)"
        },
        "G66": {
          "content": "=PIVOT.VALUE(8,\"price_subtotal\",\"#user_id\",4)"
        },
        "G67": {
          "content": "=PIVOT.VALUE(8,\"price_subtotal\",\"#user_id\",5)"
        },
        "G68": {
          "content": "=PIVOT.VALUE(8,\"price_subtotal\",\"#user_id\",6)"
        },
        "G69": {
          "content": "=PIVOT.VALUE(8,\"price_subtotal\",\"#user_id\",7)"
        },
        "G70": {
          "content": "=PIVOT.VALUE(8,\"price_subtotal\",\"#user_id\",8)"
        },
        "G71": {
          "content": "=PIVOT.VALUE(8,\"price_subtotal\",\"#user_id\",9)"
        },
        "G72": {
          "content": "=PIVOT.VALUE(8,\"price_subtotal\",\"#user_id\",10)"
        },
        "G75": { "content": "=_t(\"Revenue\")" },
        "G76": {
          "content": "=PIVOT.VALUE(10,\"price_subtotal\",\"#medium_id\",1)"
        },
        "G77": {
          "content": "=PIVOT.VALUE(10,\"price_subtotal\",\"#medium_id\",2)"
        },
        "G78": {
          "content": "=PIVOT.VALUE(10,\"price_subtotal\",\"#medium_id\",3)"
        },
        "G79": {
          "content": "=PIVOT.VALUE(10,\"price_subtotal\",\"#medium_id\",4)"
        },
        "G80": {
          "content": "=PIVOT.VALUE(10,\"price_subtotal\",\"#medium_id\",5)"
        },
        "G81": {
          "content": "=PIVOT.VALUE(10,\"price_subtotal\",\"#medium_id\",6)"
        },
        "G82": {
          "content": "=PIVOT.VALUE(10,\"price_subtotal\",\"#medium_id\",7)"
        },
        "G83": {
          "content": "=PIVOT.VALUE(10,\"price_subtotal\",\"#medium_id\",8)"
        },
        "G84": {
          "content": "=PIVOT.VALUE(10,\"price_subtotal\",\"#medium_id\",9)"
        },
        "G85": {
          "content": "=PIVOT.VALUE(10,\"price_subtotal\",\"#medium_id\",10)"
        }
      },
      "styles": {
        "A6": 1,
        "A22": 1,
        "A35": 1,
        "A48": 1,
        "A61": 1,
        "A74": 1,
        "E22": 1,
        "E35": 1,
        "E48": 1,
        "E61": 1,
        "E74": 1,
        "A23:B23": 2,
        "E23:F23": 2,
        "A24:A33": 3,
        "E24:E33": 3,
        "A36": 4,
        "A49": 4,
        "A62": 4,
        "A75": 4,
        "C36": 4,
        "C75": 4,
        "E36": 4,
        "E49": 4,
        "E62": 4,
        "E75": 4,
        "B24:C33": 5,
        "A37:C46": 5,
        "A50:C59": 5,
        "A63:C72": 5,
        "A76:C85": 5,
        "F24:G33": 5,
        "E37:G46": 5,
        "E50:G59": 5,
        "E63:G72": 5,
        "E76:G85": 5,
        "B36": 6,
        "B75": 6,
        "B49:C49": 6,
        "B62:C62": 6,
        "F36:G36": 6,
        "F49:G49": 6,
        "F62:G62": 6,
        "F75:G75": 6,
        "C23": 7,
        "G23": 7
      },
      "formats": {},
      "borders": {
        "A22:C22": 1,
        "A35:C35": 1,
        "A48:C48": 1,
        "A61:C61": 1,
        "A74:C74": 1,
        "A6:G6": 1,
        "E22:G22": 1,
        "E35:G35": 1,
        "E48:G48": 1,
        "E61:G61": 1,
        "E74:G74": 1,
        "B62": 2,
        "B75": 2,
        "E23:F23": 2,
        "F49": 2,
        "F62": 2,
        "F75": 2,
        "A7:G7": 2,
        "A23:B23": 3,
        "A24:B24": 4,
        "A25:B32": 5,
        "A37:C46": 5,
        "A50:C59": 5,
        "A64:C72": 5,
        "A77:C85": 5,
        "E25:G33": 5,
        "E37:G46": 5,
        "E51:G59": 5,
        "E64:G72": 5,
        "E77:G85": 5,
        "A33:B33": 6,
        "A34:C34": 7,
        "A36": 8,
        "A47:C47": 9,
        "A60:C60": 9,
        "A73:C73": 9,
        "A86:C86": 9,
        "E34:G34": 9,
        "E47:G47": 9,
        "E60:G60": 9,
        "E73:G73": 9,
        "E86:G86": 9,
        "A49": 10,
        "E36": 10,
        "A62": 11,
        "A75": 11,
        "E49": 11,
        "E62": 11,
        "E75": 11,
        "A63:C63": 12,
        "A76:C76": 12,
        "E24:G24": 12,
        "E50:G50": 12,
        "E63:G63": 12,
        "E76:G76": 12,
        "B36": 13,
        "B49": 13,
        "F36": 13,
        "C23": 14,
        "C24": 15,
        "C25:C32": 16,
        "C33": 17,
        "C36": 18,
        "C49": 19,
        "G36": 19,
        "C62": 20,
        "C75": 20,
        "G23": 20,
        "G49": 20,
        "G62": 20,
        "G75": 20,
        "D23": 21,
        "D24:D33": 22,
        "D36": 22,
        "D37:D46": 23,
        "D49:D59": 23,
        "D62:D72": 23,
        "D75:D85": 23
      },
      "conditionalFormats": [
        {
          "rule": {
            "type": "DataBarRule",
            "color": 16775149,
            "rangeValues": "C24:C33"
          },
          "id": "3e153fc3-0c7c-4713-991c-48f6921d7f51",
          "ranges": ["A24:A33"]
        },
        {
          "rule": {
            "type": "DataBarRule",
            "color": 15726335,
            "rangeValues": "G24:G33"
          },
          "id": "7b9e9b33-151f-4190-a0a3-9a762d831033",
          "ranges": ["E24:E33"]
        },
        {
          "rule": {
            "type": "DataBarRule",
            "color": 15531509,
            "rangeValues": "C37:C46"
          },
          "id": "947af8f8-96d3-4a02-89e7-eb1b12d95517",
          "ranges": ["A37:A46"]
        },
        {
          "rule": {
            "type": "DataBarRule",
            "color": 16708338,
            "rangeValues": "G37:G46"
          },
          "id": "33e47ff1-2748-4243-a03a-ec896dca969f",
          "ranges": ["E37:E46"]
        },
        {
          "rule": {
            "type": "DataBarRule",
            "color": 16775149,
            "rangeValues": "C50:C59"
          },
          "id": "e957d7f6-aaea-4c20-b474-e30a6949c5d1",
          "ranges": ["A50:A59"]
        },
        {
          "rule": {
            "type": "DataBarRule",
            "color": 15726335,
            "rangeValues": "G50:G59"
          },
          "id": "fc65bd3e-81a8-4490-9b87-159f1862e58f",
          "ranges": ["E50:E59"]
        },
        {
          "rule": {
            "type": "DataBarRule",
            "color": 15531509,
            "rangeValues": "C63:C72"
          },
          "id": "99da46d6-80cb-42a6-9147-4fb5e1a91dda",
          "ranges": ["A63:A72"]
        },
        {
          "rule": {
            "type": "DataBarRule",
            "color": 16708338,
            "rangeValues": "G63:G72"
          },
          "id": "a4f8bba7-132d-4542-8a69-d4ac18bdc772",
          "ranges": ["E63:E72"]
        },
        {
          "rule": {
            "type": "DataBarRule",
            "color": 16775149,
            "rangeValues": "C76:C85"
          },
          "id": "71bf3b6c-7ba1-4883-875e-b93482b51236",
          "ranges": ["A76:A85"]
        },
        {
          "rule": {
            "type": "DataBarRule",
            "color": 15726335,
            "rangeValues": "G76:G85"
          },
          "id": "f5bc51ad-eac7-4ce7-9432-d39520cbd3cb",
          "ranges": ["E76:E85"]
        }
      ],
      "figures": [
        {
          "id": "51823220-f22b-4359-8711-579a249c91bb",
          "x": 0,
          "y": 11,
          "width": 213,
          "height": 101,
          "tag": "chart",
          "data": {
            "baselineColorDown": "#DC6965",
            "baselineColorUp": "#00A04A",
            "baselineMode": "percentage",
            "title": { "text": "Quotations", "bold": true, "color": "#434343" },
            "type": "scorecard",
            "background": "#EFF6FF",
            "baseline": "Data!E4",
            "baselineDescr": "since last period",
            "keyValue": "Data!D4",
            "humanize": false
          }
        },
        {
          "id": "9a38934c-b454-4a4b-88aa-17d1b80dbf5f",
          "x": 223,
          "y": 11,
          "width": 211,
          "height": 101,
          "tag": "chart",
          "data": {
            "baselineColorDown": "#DC6965",
            "baselineColorUp": "#00A04A",
            "baselineMode": "percentage",
            "title": { "text": "Orders", "color": "#434343", "bold": true },
            "type": "scorecard",
            "background": "#EFF6FF",
            "baseline": "Data!E5",
            "baselineDescr": "since last period",
            "keyValue": "Data!D5",
            "humanize": false
          }
        },
        {
          "id": "67858d0e-b5ba-4a3c-bf9e-c0fceaeedf65",
          "x": 444,
          "y": 11,
          "width": 218,
          "height": 101,
          "tag": "chart",
          "data": {
            "baselineColorDown": "#DC6965",
            "baselineColorUp": "#00A04A",
            "baselineMode": "percentage",
            "title": { "text": "Revenue", "color": "#434343", "bold": true },
            "type": "scorecard",
            "background": "#FFF7ED",
            "baseline": "Data!E7",
            "baselineDescr": "since last period",
            "keyValue": "Data!D7",
            "humanize": false
          }
        },
        {
          "id": "d43375c1-73a6-42a2-8dbd-0f13c285824f",
          "x": 672,
          "y": 11,
          "width": 213,
          "height": 101,
          "tag": "chart",
          "data": {
            "baselineColorDown": "#DC6965",
            "baselineColorUp": "#00A04A",
            "baselineMode": "percentage",
            "title": {
              "text": "Average Order",
              "color": "#434343",
              "bold": true
            },
            "type": "scorecard",
            "background": "#FFF7ED",
            "baseline": "Data!E8",
            "baselineDescr": "since last period",
            "keyValue": "Data!D8",
            "humanize": false
          }
        },
        {
          "id": "a527960b-0812-4291-baba-f6b4b5280a0d",
          "x": 0,
          "y": 155,
          "width": 1095,
          "height": 344,
          "tag": "chart",
          "data": {
            "title": { "text": "" },
            "background": "#FFFFFF",
            "legendPosition": "none",
            "metaData": {
              "groupBy": ["date:month"],
              "measure": "price_subtotal",
              "order": null,
              "resModel": "sale.report",
              "mode": "line"
            },
            "searchParams": {
              "comparison": null,
              "context": { "group_by": [] },
              "domain": [["state", "not in", ["draft", "cancel", "sent"]]],
              "groupBy": ["date:month"],
              "orderBy": []
            },
            "type": "odoo_line",
            "verticalAxisPosition": "left",
            "stacked": false,
            "fillArea": true,
            "fieldMatching": {
              "13d30fda-b14d-4a56-b186-25468af3b1e9": {
                "chain": "date",
                "type": "datetime",
                "offset": 0
              },
              "e6db018b-19ec-42c3-b29e-11b1e3910916": {
                "chain": "country_id",
                "type": "many2one"
              },
              "dbb716a1-5977-47ee-a53e-ca5716303433": {
                "chain": "product_id",
                "type": "many2one"
              },
              "7d95fe28-fd13-4805-948c-95742e7d6733": {
                "chain": "partner_id",
                "type": "many2one"
              },
              "27e323f4-cd70-4345-82fa-dfc855707bc1": {
                "chain": "categ_id",
                "type": "many2one"
              },
              "b75963ca-ab5f-4da5-9c90-67526517e5e7": {
                "chain": "team_id",
                "type": "many2one"
              },
              "4181e8e3-2e7e-42be-a88b-f24acb7d03e7": {
                "chain": "user_id",
                "type": "many2one"
              },
              "ec466626-f54a-4955-9611-eaa2719f1afb": {
                "chain": "source_id",
                "type": "many2one"
              },
              "85d94827-6ce2-429c-9775-ef646cfddb6b": {
                "chain": "medium_id",
                "type": "many2one"
              }
            }
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
      "id": "eae01f9c-c461-4489-ade4-957ef2459d40",
      "name": "Data",
      "colNumber": 26,
      "rowNumber": 103,
      "rows": {},
      "cols": {
        "0": { "size": 160 },
        "1": { "size": 89 },
        "2": { "size": 89 },
        "3": { "size": 89 },
        "4": { "size": 89 }
      },
      "merges": [],
      "cells": {
        "A1": { "content": "=_t(\"KPI\")" },
        "A2": { "content": "=_t(\"Draft quotations\")" },
        "A3": { "content": "=_t(\"Quotations sent\")" },
        "A4": { "content": "=_t(\"Total quotations\")" },
        "A5": { "content": "=_t(\"Orders\")" },
        "A6": { "content": "=_t(\"Total orders\")" },
        "A7": { "content": "=_t(\"Revenue\")" },
        "A8": { "content": "=_t(\"Average order amount\")" },
        "B1": { "content": "=_t(\"Current\")" },
        "B2": {
          "content": "=PIVOT.VALUE(11,\"order_reference\",\"state\",\"draft\")"
        },
        "B3": {
          "content": "=PIVOT.VALUE(11,\"order_reference\",\"state\",\"sent\")"
        },
        "B4": { "content": "=B2+B3" },
        "B5": {
          "content": "=PIVOT.VALUE(11,\"order_reference\",\"state\",\"sale\")"
        },
        "B6": {
          "content": "=PIVOT.VALUE(11,\"order_reference\",\"state\",\"sale\")"
        },
        "B7": {
          "content": "=PIVOT.VALUE(11,\"price_subtotal\",\"state\",\"sale\")"
        },
        "B8": { "content": "=IFERROR(B7/B6)" },
        "C1": { "content": "=_t(\"Previous\")" },
        "C2": {
          "content": "=PIVOT.VALUE(12,\"order_reference\",\"state\",\"draft\")"
        },
        "C3": {
          "content": "=PIVOT.VALUE(12,\"order_reference\",\"state\",\"sent\")"
        },
        "C4": { "content": "=C2+C3" },
        "C5": {
          "content": "=PIVOT.VALUE(12,\"order_reference\",\"state\",\"sale\")"
        },
        "C6": {
          "content": "=PIVOT.VALUE(12,\"order_reference\",\"state\",\"sale\")"
        },
        "C7": {
          "content": "=PIVOT.VALUE(12,\"price_subtotal\",\"state\",\"sale\")"
        },
        "C8": { "content": "=IFERROR(C7/C6)" },
        "D1": { "content": "=_t(\"Current\")" },
        "D2": { "content": "=FORMAT.LARGE.NUMBER(B2)" },
        "D3": { "content": "=FORMAT.LARGE.NUMBER(B3)" },
        "D4": { "content": "=FORMAT.LARGE.NUMBER(B4)" },
        "D5": { "content": "=FORMAT.LARGE.NUMBER(B5)" },
        "D6": { "content": "=FORMAT.LARGE.NUMBER(B6)" },
        "D7": { "content": "=FORMAT.LARGE.NUMBER(B7)" },
        "D8": { "content": "=FORMAT.LARGE.NUMBER(B8)" },
        "E1": { "content": "=_t(\"Previous\")" },
        "E2": { "content": "=FORMAT.LARGE.NUMBER(C2)" },
        "E3": { "content": "=FORMAT.LARGE.NUMBER(C3)" },
        "E4": { "content": "=FORMAT.LARGE.NUMBER(C4)" },
        "E5": { "content": "=FORMAT.LARGE.NUMBER(C5)" },
        "E6": { "content": "=FORMAT.LARGE.NUMBER(C6)" },
        "E7": { "content": "=FORMAT.LARGE.NUMBER(C7)" },
        "E8": { "content": "=FORMAT.LARGE.NUMBER(C8)" }
      },
      "styles": { "A1:E1": 8, "D2:E8": 9 },
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
    "1": { "textColor": "#01666b", "bold": true, "fontSize": 16 },
    "2": {
      "fontSize": 11,
      "textColor": "#434343",
      "verticalAlign": "middle",
      "bold": true
    },
    "3": { "textColor": "#01666B", "verticalAlign": "middle" },
    "4": { "bold": true, "fontSize": 11, "textColor": "#434343" },
    "5": { "textColor": "#434343", "verticalAlign": "middle" },
    "6": {
      "bold": true,
      "fontSize": 11,
      "textColor": "#434343",
      "align": "center"
    },
    "7": {
      "align": "center",
      "fontSize": 11,
      "textColor": "#434343",
      "verticalAlign": "middle",
      "bold": true
    },
    "8": { "bold": true },
    "9": { "fillColor": "#f2f2f2" }
  },
  "formats": {},
  "borders": {
    "1": { "bottom": { "style": "thin", "color": "#CCCCCC" } },
    "2": { "top": { "style": "thin", "color": "#CCCCCC" } },
    "3": {
      "top": { "style": "thin", "color": "#CCCCCC" },
      "bottom": { "style": "thin", "color": "#FFFFFF" }
    },
    "4": {
      "top": { "style": "thin", "color": "#FFFFFF" },
      "bottom": { "style": "thick", "color": "#FFFFFF" },
      "left": { "style": "thick", "color": "#FFFFFF" },
      "right": { "style": "thick", "color": "#FFFFFF" }
    },
    "5": {
      "top": { "style": "thick", "color": "#FFFFFF" },
      "bottom": { "style": "thick", "color": "#FFFFFF" },
      "left": { "style": "thick", "color": "#FFFFFF" },
      "right": { "style": "thick", "color": "#FFFFFF" }
    },
    "6": {
      "top": { "style": "thick", "color": "#FFFFFF" },
      "bottom": { "style": "thin", "color": "#FFFFFF" },
      "left": { "style": "thick", "color": "#FFFFFF" },
      "right": { "style": "thick", "color": "#FFFFFF" }
    },
    "7": { "top": { "style": "thin", "color": "#FFFFFF" } },
    "8": {
      "top": { "style": "thin", "color": "#CCCCCC" },
      "bottom": { "style": "thick", "color": "#FFFFFF" },
      "left": { "style": "thin", "color": "#FFFFFF" }
    },
    "9": { "top": { "style": "thick", "color": "#FFFFFF" } },
    "10": {
      "top": { "style": "thin", "color": "#CCCCCC" },
      "bottom": { "style": "thick", "color": "#FFFFFF" },
      "left": { "style": "thick", "color": "#FFFFFF" }
    },
    "11": {
      "top": { "style": "thin", "color": "#CCCCCC" },
      "left": { "style": "thick", "color": "#FFFFFF" }
    },
    "12": {
      "bottom": { "style": "thick", "color": "#FFFFFF" },
      "left": { "style": "thick", "color": "#FFFFFF" },
      "right": { "style": "thick", "color": "#FFFFFF" }
    },
    "13": {
      "top": { "style": "thin", "color": "#CCCCCC" },
      "bottom": { "style": "thick", "color": "#FFFFFF" }
    },
    "14": {
      "top": { "style": "thin", "color": "#CCCCCC" },
      "bottom": { "style": "thin", "color": "#FFFFFF" },
      "right": { "style": "thin", "color": "#FFFFFF" }
    },
    "15": {
      "top": { "style": "thin", "color": "#FFFFFF" },
      "bottom": { "style": "thick", "color": "#FFFFFF" },
      "left": { "style": "thick", "color": "#FFFFFF" },
      "right": { "style": "thin", "color": "#FFFFFF" }
    },
    "16": {
      "top": { "style": "thick", "color": "#FFFFFF" },
      "bottom": { "style": "thick", "color": "#FFFFFF" },
      "left": { "style": "thick", "color": "#FFFFFF" },
      "right": { "style": "thin", "color": "#FFFFFF" }
    },
    "17": {
      "top": { "style": "thick", "color": "#FFFFFF" },
      "bottom": { "style": "thin", "color": "#FFFFFF" },
      "left": { "style": "thick", "color": "#FFFFFF" },
      "right": { "style": "thin", "color": "#FFFFFF" }
    },
    "18": {
      "top": { "style": "thin", "color": "#CCCCCC" },
      "bottom": { "style": "thick", "color": "#FFFFFF" },
      "right": { "style": "thin", "color": "#FFFFFF" }
    },
    "19": {
      "top": { "style": "thin", "color": "#CCCCCC" },
      "bottom": { "style": "thick", "color": "#FFFFFF" },
      "right": { "style": "thick", "color": "#FFFFFF" }
    },
    "20": {
      "top": { "style": "thin", "color": "#CCCCCC" },
      "right": { "style": "thick", "color": "#FFFFFF" }
    },
    "21": { "left": { "style": "thin", "color": "#FFFFFF" } },
    "22": {
      "left": { "style": "thin", "color": "#FFFFFF" },
      "right": { "style": "thick", "color": "#FFFFFF" }
    },
    "23": {
      "left": { "style": "thick", "color": "#FFFFFF" },
      "right": { "style": "thick", "color": "#FFFFFF" }
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
    "3": {
      "type": "ODOO",
      "fieldMatching": {
        "13d30fda-b14d-4a56-b186-25468af3b1e9": {
          "chain": "date",
          "type": "datetime",
          "offset": 0
        },
        "e6db018b-19ec-42c3-b29e-11b1e3910916": {
          "chain": "country_id",
          "type": "many2one"
        },
        "dbb716a1-5977-47ee-a53e-ca5716303433": {
          "chain": "product_id",
          "type": "many2one"
        },
        "7d95fe28-fd13-4805-948c-95742e7d6733": {
          "chain": "partner_id",
          "type": "many2one"
        },
        "27e323f4-cd70-4345-82fa-dfc855707bc1": {
          "chain": "categ_id",
          "type": "many2one"
        },
        "b75963ca-ab5f-4da5-9c90-67526517e5e7": {
          "chain": "team_id",
          "type": "many2one"
        },
        "4181e8e3-2e7e-42be-a88b-f24acb7d03e7": {
          "chain": "user_id",
          "type": "many2one"
        },
        "ec466626-f54a-4955-9611-eaa2719f1afb": {
          "chain": "source_id",
          "type": "many2one"
        },
        "85d94827-6ce2-429c-9775-ef646cfddb6b": {
          "chain": "medium_id",
          "type": "many2one"
        }
      },
      "context": {
        "params": {
          "action": 1275,
          "model": "sale.report",
          "view_type": "pivot",
          "menu_id": 878,
          "cids": 1
        },
        "group_by": []
      },
      "domain": [
        "&",
        ["categ_id", "!=", false],
        ["state", "not in", ["draft", "sent", "cancel"]]
      ],
      "id": "3",
      "measures": [
        { "id": "order_reference", "fieldName": "order_reference" },
        { "id": "price_subtotal", "fieldName": "price_subtotal" }
      ],
      "model": "sale.report",
      "name": "Sales Analysis by Product Category",
      "sortedColumn": {
        "groupId": [[], []],
        "measure": "price_subtotal",
        "order": "desc"
      },
      "formulaId": "3",
      "columns": [],
      "rows": [{ "fieldName": "categ_id" }]
    },
    "4": {
      "type": "ODOO",
      "fieldMatching": {
        "13d30fda-b14d-4a56-b186-25468af3b1e9": {
          "chain": "date",
          "type": "datetime",
          "offset": 0
        },
        "e6db018b-19ec-42c3-b29e-11b1e3910916": {
          "chain": "country_id",
          "type": "many2one"
        },
        "dbb716a1-5977-47ee-a53e-ca5716303433": {
          "chain": "product_id",
          "type": "many2one"
        },
        "7d95fe28-fd13-4805-948c-95742e7d6733": {
          "chain": "partner_id",
          "type": "many2one"
        },
        "27e323f4-cd70-4345-82fa-dfc855707bc1": {
          "chain": "categ_id",
          "type": "many2one"
        },
        "b75963ca-ab5f-4da5-9c90-67526517e5e7": {
          "chain": "team_id",
          "type": "many2one"
        },
        "4181e8e3-2e7e-42be-a88b-f24acb7d03e7": {
          "chain": "user_id",
          "type": "many2one"
        },
        "ec466626-f54a-4955-9611-eaa2719f1afb": {
          "chain": "source_id",
          "type": "many2one"
        },
        "85d94827-6ce2-429c-9775-ef646cfddb6b": {
          "chain": "medium_id",
          "type": "many2one"
        }
      },
      "context": {
        "params": {
          "action": 1275,
          "model": "sale.report",
          "view_type": "pivot",
          "menu_id": 878,
          "cids": 1
        },
        "group_by": []
      },
      "domain": [
        "&",
        ["partner_id", "!=", false],
        ["state", "not in", ["draft", "sent", "cancel"]]
      ],
      "id": "4",
      "measures": [
        { "id": "order_reference", "fieldName": "order_reference" },
        { "id": "price_subtotal", "fieldName": "price_subtotal" }
      ],
      "model": "sale.report",
      "name": "Sales Analysis by Customer",
      "sortedColumn": {
        "groupId": [[], []],
        "measure": "price_subtotal",
        "order": "desc"
      },
      "formulaId": "4",
      "columns": [],
      "rows": [{ "fieldName": "partner_id" }]
    },
    "5": {
      "type": "ODOO",
      "fieldMatching": {
        "13d30fda-b14d-4a56-b186-25468af3b1e9": {
          "chain": "date",
          "type": "datetime",
          "offset": 0
        },
        "e6db018b-19ec-42c3-b29e-11b1e3910916": {
          "chain": "country_id",
          "type": "many2one"
        },
        "dbb716a1-5977-47ee-a53e-ca5716303433": {
          "chain": "product_id",
          "type": "many2one"
        },
        "7d95fe28-fd13-4805-948c-95742e7d6733": {
          "chain": "partner_id",
          "type": "many2one"
        },
        "27e323f4-cd70-4345-82fa-dfc855707bc1": {
          "chain": "categ_id",
          "type": "many2one"
        },
        "b75963ca-ab5f-4da5-9c90-67526517e5e7": {
          "chain": "team_id",
          "type": "many2one"
        },
        "4181e8e3-2e7e-42be-a88b-f24acb7d03e7": {
          "chain": "user_id",
          "type": "many2one"
        },
        "ec466626-f54a-4955-9611-eaa2719f1afb": {
          "chain": "source_id",
          "type": "many2one"
        },
        "85d94827-6ce2-429c-9775-ef646cfddb6b": {
          "chain": "medium_id",
          "type": "many2one"
        }
      },
      "context": {
        "params": {
          "action": 1275,
          "model": "sale.report",
          "view_type": "pivot",
          "menu_id": 878,
          "cids": 1
        },
        "group_by": []
      },
      "domain": [
        "&",
        ["country_id", "!=", false],
        ["state", "not in", ["draft", "sent", "cancel"]]
      ],
      "id": "5",
      "measures": [
        { "id": "order_reference", "fieldName": "order_reference" },
        { "id": "price_subtotal", "fieldName": "price_subtotal" }
      ],
      "model": "sale.report",
      "name": "Sales Analysis by Customer Country",
      "sortedColumn": {
        "groupId": [[], []],
        "measure": "price_subtotal",
        "order": "desc"
      },
      "formulaId": "5",
      "columns": [],
      "rows": [{ "fieldName": "country_id" }]
    },
    "6": {
      "type": "ODOO",
      "fieldMatching": {
        "13d30fda-b14d-4a56-b186-25468af3b1e9": {
          "chain": "date",
          "type": "datetime",
          "offset": 0
        },
        "e6db018b-19ec-42c3-b29e-11b1e3910916": {
          "chain": "country_id",
          "type": "many2one"
        },
        "dbb716a1-5977-47ee-a53e-ca5716303433": {
          "chain": "product_id",
          "type": "many2one"
        },
        "7d95fe28-fd13-4805-948c-95742e7d6733": {
          "chain": "partner_id",
          "type": "many2one"
        },
        "27e323f4-cd70-4345-82fa-dfc855707bc1": {
          "chain": "categ_id",
          "type": "many2one"
        },
        "b75963ca-ab5f-4da5-9c90-67526517e5e7": {
          "chain": "team_id",
          "type": "many2one"
        },
        "4181e8e3-2e7e-42be-a88b-f24acb7d03e7": {
          "chain": "user_id",
          "type": "many2one"
        },
        "ec466626-f54a-4955-9611-eaa2719f1afb": {
          "chain": "source_id",
          "type": "many2one"
        },
        "85d94827-6ce2-429c-9775-ef646cfddb6b": {
          "chain": "medium_id",
          "type": "many2one"
        }
      },
      "context": {
        "params": {
          "action": 1275,
          "model": "sale.report",
          "view_type": "pivot",
          "menu_id": 878,
          "cids": 1
        },
        "group_by": []
      },
      "domain": [
        "&",
        ["product_tmpl_id", "!=", false],
        ["state", "not in", ["draft", "sent", "cancel"]]
      ],
      "id": "6",
      "measures": [
        { "id": "order_reference", "fieldName": "order_reference" },
        { "id": "price_subtotal", "fieldName": "price_subtotal" }
      ],
      "model": "sale.report",
      "name": "Sales Analysis by Product Variant",
      "sortedColumn": {
        "groupId": [[], []],
        "measure": "price_subtotal",
        "order": "desc"
      },
      "formulaId": "6",
      "columns": [],
      "rows": [{ "fieldName": "product_id" }]
    },
    "7": {
      "type": "ODOO",
      "fieldMatching": {
        "13d30fda-b14d-4a56-b186-25468af3b1e9": {
          "chain": "date",
          "type": "datetime",
          "offset": 0
        },
        "e6db018b-19ec-42c3-b29e-11b1e3910916": {
          "chain": "country_id",
          "type": "many2one"
        },
        "dbb716a1-5977-47ee-a53e-ca5716303433": {
          "chain": "product_id",
          "type": "many2one"
        },
        "7d95fe28-fd13-4805-948c-95742e7d6733": {
          "chain": "partner_id",
          "type": "many2one"
        },
        "27e323f4-cd70-4345-82fa-dfc855707bc1": {
          "chain": "categ_id",
          "type": "many2one"
        },
        "b75963ca-ab5f-4da5-9c90-67526517e5e7": {
          "chain": "team_id",
          "type": "many2one"
        },
        "4181e8e3-2e7e-42be-a88b-f24acb7d03e7": {
          "chain": "user_id",
          "type": "many2one"
        },
        "ec466626-f54a-4955-9611-eaa2719f1afb": {
          "chain": "source_id",
          "type": "many2one"
        },
        "85d94827-6ce2-429c-9775-ef646cfddb6b": {
          "chain": "medium_id",
          "type": "many2one"
        }
      },
      "context": { "group_by": [] },
      "domain": [
        "&",
        ["team_id", "!=", false],
        ["state", "not in", ["draft", "sent", "cancel"]]
      ],
      "id": "7",
      "measures": [
        { "id": "order_reference", "fieldName": "order_reference" },
        { "id": "price_subtotal", "fieldName": "price_subtotal" }
      ],
      "model": "sale.report",
      "name": "Sales Analysis by Sales Team",
      "sortedColumn": {
        "groupId": [[], []],
        "measure": "price_subtotal",
        "order": "desc"
      },
      "formulaId": "7",
      "columns": [],
      "rows": [{ "fieldName": "team_id" }]
    },
    "8": {
      "type": "ODOO",
      "fieldMatching": {
        "13d30fda-b14d-4a56-b186-25468af3b1e9": {
          "chain": "date",
          "type": "datetime",
          "offset": 0
        },
        "e6db018b-19ec-42c3-b29e-11b1e3910916": {
          "chain": "country_id",
          "type": "many2one"
        },
        "dbb716a1-5977-47ee-a53e-ca5716303433": {
          "chain": "product_id",
          "type": "many2one"
        },
        "7d95fe28-fd13-4805-948c-95742e7d6733": {
          "chain": "partner_id",
          "type": "many2one"
        },
        "27e323f4-cd70-4345-82fa-dfc855707bc1": {
          "chain": "categ_id",
          "type": "many2one"
        },
        "b75963ca-ab5f-4da5-9c90-67526517e5e7": {
          "chain": "team_id",
          "type": "many2one"
        },
        "4181e8e3-2e7e-42be-a88b-f24acb7d03e7": {
          "chain": "user_id",
          "type": "many2one"
        },
        "ec466626-f54a-4955-9611-eaa2719f1afb": {
          "chain": "source_id",
          "type": "many2one"
        },
        "85d94827-6ce2-429c-9775-ef646cfddb6b": {
          "chain": "medium_id",
          "type": "many2one"
        }
      },
      "context": { "group_by": [] },
      "domain": [
        "&",
        ["user_id", "!=", false],
        ["state", "not in", ["draft", "sent", "cancel"]]
      ],
      "id": "8",
      "measures": [
        { "id": "order_reference", "fieldName": "order_reference" },
        { "id": "price_subtotal", "fieldName": "price_subtotal" }
      ],
      "model": "sale.report",
      "name": "Sales Analysis by Salesperson",
      "sortedColumn": {
        "groupId": [[], []],
        "measure": "price_subtotal",
        "order": "desc"
      },
      "formulaId": "8",
      "columns": [],
      "rows": [{ "fieldName": "user_id" }]
    },
    "9": {
      "type": "ODOO",
      "fieldMatching": {
        "13d30fda-b14d-4a56-b186-25468af3b1e9": {
          "chain": "date",
          "type": "datetime",
          "offset": 0
        },
        "e6db018b-19ec-42c3-b29e-11b1e3910916": {
          "chain": "country_id",
          "type": "many2one"
        },
        "dbb716a1-5977-47ee-a53e-ca5716303433": {
          "chain": "product_id",
          "type": "many2one"
        },
        "7d95fe28-fd13-4805-948c-95742e7d6733": {
          "chain": "partner_id",
          "type": "many2one"
        },
        "27e323f4-cd70-4345-82fa-dfc855707bc1": {
          "chain": "categ_id",
          "type": "many2one"
        },
        "b75963ca-ab5f-4da5-9c90-67526517e5e7": {
          "chain": "team_id",
          "type": "many2one"
        },
        "4181e8e3-2e7e-42be-a88b-f24acb7d03e7": {
          "chain": "user_id",
          "type": "many2one"
        },
        "ec466626-f54a-4955-9611-eaa2719f1afb": {
          "chain": "source_id",
          "type": "many2one"
        },
        "85d94827-6ce2-429c-9775-ef646cfddb6b": {
          "chain": "medium_id",
          "type": "many2one"
        }
      },
      "context": { "group_by": [] },
      "domain": [
        "&",
        ["source_id", "!=", false],
        ["state", "not in", ["draft", "sent", "cancel"]]
      ],
      "id": "9",
      "measures": [
        { "id": "order_reference", "fieldName": "order_reference" },
        { "id": "price_subtotal", "fieldName": "price_subtotal" }
      ],
      "model": "sale.report",
      "name": "Sales Analysis by Source",
      "sortedColumn": {
        "groupId": [[], []],
        "measure": "price_subtotal",
        "order": "desc"
      },
      "formulaId": "9",
      "columns": [],
      "rows": [{ "fieldName": "source_id" }]
    },
    "10": {
      "type": "ODOO",
      "fieldMatching": {
        "13d30fda-b14d-4a56-b186-25468af3b1e9": {
          "chain": "date",
          "type": "datetime",
          "offset": 0
        },
        "e6db018b-19ec-42c3-b29e-11b1e3910916": {
          "chain": "country_id",
          "type": "many2one"
        },
        "dbb716a1-5977-47ee-a53e-ca5716303433": {
          "chain": "product_id",
          "type": "many2one"
        },
        "7d95fe28-fd13-4805-948c-95742e7d6733": {
          "chain": "partner_id",
          "type": "many2one"
        },
        "27e323f4-cd70-4345-82fa-dfc855707bc1": {
          "chain": "categ_id",
          "type": "many2one"
        },
        "b75963ca-ab5f-4da5-9c90-67526517e5e7": {
          "chain": "team_id",
          "type": "many2one"
        },
        "4181e8e3-2e7e-42be-a88b-f24acb7d03e7": {
          "chain": "user_id",
          "type": "many2one"
        },
        "ec466626-f54a-4955-9611-eaa2719f1afb": {
          "chain": "source_id",
          "type": "many2one"
        },
        "85d94827-6ce2-429c-9775-ef646cfddb6b": {
          "chain": "medium_id",
          "type": "many2one"
        }
      },
      "context": { "group_by": [] },
      "domain": [
        "&",
        ["medium_id", "!=", false],
        ["state", "not in", ["draft", "sent", "cancel"]]
      ],
      "id": "10",
      "measures": [
        { "id": "order_reference", "fieldName": "order_reference" },
        { "id": "price_subtotal", "fieldName": "price_subtotal" }
      ],
      "model": "sale.report",
      "name": "Sales Analysis by Medium",
      "sortedColumn": {
        "groupId": [[], []],
        "measure": "price_subtotal",
        "order": "desc"
      },
      "formulaId": "10",
      "columns": [],
      "rows": [{ "fieldName": "medium_id" }]
    },
    "11": {
      "type": "ODOO",
      "fieldMatching": {
        "13d30fda-b14d-4a56-b186-25468af3b1e9": {
          "chain": "date",
          "type": "datetime",
          "offset": 0
        },
        "e6db018b-19ec-42c3-b29e-11b1e3910916": {
          "chain": "partner_id.country_id",
          "type": "many2one"
        },
        "dbb716a1-5977-47ee-a53e-ca5716303433": {
          "chain": "product_id",
          "type": "many2one"
        },
        "7d95fe28-fd13-4805-948c-95742e7d6733": {
          "chain": "partner_id",
          "type": "many2one"
        },
        "27e323f4-cd70-4345-82fa-dfc855707bc1": {
          "chain": "product_id.categ_id",
          "type": "many2one"
        },
        "b75963ca-ab5f-4da5-9c90-67526517e5e7": {
          "chain": "team_id",
          "type": "many2one"
        },
        "4181e8e3-2e7e-42be-a88b-f24acb7d03e7": {
          "chain": "user_id",
          "type": "many2one"
        },
        "ec466626-f54a-4955-9611-eaa2719f1afb": {
          "chain": "source_id",
          "type": "many2one"
        },
        "85d94827-6ce2-429c-9775-ef646cfddb6b": {
          "chain": "medium_id",
          "type": "many2one"
        }
      },
      "context": {},
      "domain": [["state", "in", ["draft", "sent", "sale"]]],
      "id": "11",
      "measures": [
        { "id": "order_reference", "fieldName": "order_reference" },
        { "id": "price_subtotal", "fieldName": "price_subtotal" }
      ],
      "model": "sale.report",
      "name": "so stats - current",
      "sortedColumn": null,
      "formulaId": "11",
      "columns": [],
      "rows": [{ "fieldName": "state" }]
    },
    "12": {
      "type": "ODOO",
      "fieldMatching": {
        "13d30fda-b14d-4a56-b186-25468af3b1e9": {
          "chain": "date",
          "type": "datetime",
          "offset": -1
        },
        "e6db018b-19ec-42c3-b29e-11b1e3910916": {
          "chain": "partner_id.country_id",
          "type": "many2one"
        },
        "dbb716a1-5977-47ee-a53e-ca5716303433": {
          "chain": "product_id",
          "type": "many2one"
        },
        "7d95fe28-fd13-4805-948c-95742e7d6733": {
          "chain": "partner_id",
          "type": "many2one"
        },
        "27e323f4-cd70-4345-82fa-dfc855707bc1": {
          "chain": "product_id.categ_id",
          "type": "many2one"
        },
        "b75963ca-ab5f-4da5-9c90-67526517e5e7": {
          "chain": "team_id",
          "type": "many2one"
        },
        "4181e8e3-2e7e-42be-a88b-f24acb7d03e7": {
          "chain": "user_id",
          "type": "many2one"
        },
        "ec466626-f54a-4955-9611-eaa2719f1afb": {
          "chain": "source_id",
          "type": "many2one"
        },
        "85d94827-6ce2-429c-9775-ef646cfddb6b": {
          "chain": "medium_id",
          "type": "many2one"
        }
      },
      "context": {},
      "domain": [["state", "in", ["draft", "sent", "sale"]]],
      "id": "12",
      "measures": [
        { "id": "order_reference", "fieldName": "order_reference" },
        { "id": "price_subtotal", "fieldName": "price_subtotal" }
      ],
      "model": "sale.report",
      "name": "so stats - previous",
      "sortedColumn": null,
      "formulaId": "12",
      "columns": [],
      "rows": [{ "fieldName": "state" }]
    }
  },
  "pivotNextId": 13,
  "customTableStyles": {},
  "odooVersion": 12,
  "globalFilters": [
    {
      "id": "13d30fda-b14d-4a56-b186-25468af3b1e9",
      "type": "date",
      "label": "Period",
      "defaultValue": "last_three_months",
      "rangeType": "relative"
    },
    {
      "id": "e6db018b-19ec-42c3-b29e-11b1e3910916",
      "type": "relation",
      "label": "Country",
      "modelName": "res.country",
      "defaultValue": [],
      "defaultValueDisplayNames": [],
      "rangeType": "year"
    },
    {
      "id": "dbb716a1-5977-47ee-a53e-ca5716303433",
      "type": "relation",
      "label": "Product",
      "modelName": "product.product",
      "defaultValue": [],
      "defaultValueDisplayNames": [],
      "rangeType": "year"
    },
    {
      "id": "7d95fe28-fd13-4805-948c-95742e7d6733",
      "type": "relation",
      "label": "Customer",
      "modelName": "res.partner",
      "defaultValue": [],
      "defaultValueDisplayNames": [],
      "rangeType": "year"
    },
    {
      "id": "27e323f4-cd70-4345-82fa-dfc855707bc1",
      "type": "relation",
      "label": "Category",
      "modelName": "product.category",
      "defaultValue": [],
      "defaultValueDisplayNames": [],
      "rangeType": "year"
    },
    {
      "id": "b75963ca-ab5f-4da5-9c90-67526517e5e7",
      "type": "relation",
      "label": "Sales Team",
      "modelName": "crm.team",
      "defaultValue": [],
      "defaultValueDisplayNames": [],
      "rangeType": "year"
    },
    {
      "id": "4181e8e3-2e7e-42be-a88b-f24acb7d03e7",
      "type": "relation",
      "label": "Salesperson",
      "modelName": "res.users",
      "defaultValue": [],
      "defaultValueDisplayNames": [],
      "rangeType": "year"
    },
    {
      "id": "ec466626-f54a-4955-9611-eaa2719f1afb",
      "type": "relation",
      "label": "Source",
      "modelName": "utm.source",
      "defaultValue": [],
      "defaultValueDisplayNames": [],
      "rangeType": "year"
    },
    {
      "id": "85d94827-6ce2-429c-9775-ef646cfddb6b",
      "type": "relation",
      "label": "Medium",
      "modelName": "utm.medium",
      "defaultValue": [],
      "defaultValueDisplayNames": [],
      "rangeType": "year"
    }
  ],
  "lists": {
    "1": {
      "columns": ["name", "partner_id", "user_id", "amount_untaxed"],
      "domain": ["|", ["state", "=", "draft"], ["state", "=", "sent"]],
      "model": "sale.order",
      "context": {},
      "orderBy": [{ "name": "amount_untaxed", "asc": false }],
      "id": "1",
      "name": "Quotations by Untaxed Amount",
      "fieldMatching": {
        "13d30fda-b14d-4a56-b186-25468af3b1e9": {
          "chain": "date_order",
          "type": "datetime",
          "offset": 0
        },
        "e6db018b-19ec-42c3-b29e-11b1e3910916": {
          "chain": "partner_id.country_id",
          "type": "many2one"
        },
        "dbb716a1-5977-47ee-a53e-ca5716303433": {
          "chain": "order_line.product_id",
          "type": "many2one"
        },
        "7d95fe28-fd13-4805-948c-95742e7d6733": {
          "chain": "partner_id",
          "type": "many2one"
        },
        "27e323f4-cd70-4345-82fa-dfc855707bc1": {
          "chain": "order_line.product_id.categ_id",
          "type": "many2one"
        },
        "b75963ca-ab5f-4da5-9c90-67526517e5e7": {
          "chain": "team_id",
          "type": "many2one"
        },
        "4181e8e3-2e7e-42be-a88b-f24acb7d03e7": {
          "chain": "user_id",
          "type": "many2one"
        },
        "ec466626-f54a-4955-9611-eaa2719f1afb": {
          "chain": "source_id",
          "type": "many2one"
        },
        "85d94827-6ce2-429c-9775-ef646cfddb6b": {
          "chain": "medium_id",
          "type": "many2one"
        }
      }
    },
    "2": {
      "columns": ["name", "partner_id", "user_id", "amount_untaxed"],
      "domain": [["state", "not in", ["draft", "sent", "cancel"]]],
      "model": "sale.order",
      "context": {},
      "orderBy": [
        { "name": "amount_untaxed", "asc": false },
        { "name": "invoice_status", "asc": true }
      ],
      "id": "2",
      "name": "Sales Orders by Untaxed Amount",
      "fieldMatching": {
        "13d30fda-b14d-4a56-b186-25468af3b1e9": {
          "chain": "date_order",
          "type": "datetime",
          "offset": 0
        },
        "e6db018b-19ec-42c3-b29e-11b1e3910916": {
          "chain": "partner_id.country_id",
          "type": "many2one"
        },
        "dbb716a1-5977-47ee-a53e-ca5716303433": {
          "chain": "order_line.product_id",
          "type": "many2one"
        },
        "7d95fe28-fd13-4805-948c-95742e7d6733": {
          "chain": "partner_id",
          "type": "many2one"
        },
        "27e323f4-cd70-4345-82fa-dfc855707bc1": {
          "chain": "order_line.product_id.categ_id",
          "type": "many2one"
        },
        "b75963ca-ab5f-4da5-9c90-67526517e5e7": {
          "chain": "team_id",
          "type": "many2one"
        },
        "4181e8e3-2e7e-42be-a88b-f24acb7d03e7": {
          "chain": "user_id",
          "type": "many2one"
        },
        "ec466626-f54a-4955-9611-eaa2719f1afb": {
          "chain": "source_id",
          "type": "many2one"
        },
        "85d94827-6ce2-429c-9775-ef646cfddb6b": {
          "chain": "medium_id",
          "type": "many2one"
        }
      }
    }
  },
  "listNextId": 3,
  "chartOdooMenusReferences": {
    "a527960b-0812-4291-baba-f6b4b5280a0d": "sale.menu_sale_order",
    "51823220-f22b-4359-8711-579a249c91bb": "sale.menu_sale_quotations",
    "9a38934c-b454-4a4b-88aa-17d1b80dbf5f": "sale.menu_sale_order",
    "67858d0e-b5ba-4a3c-bf9e-c0fceaeedf65": "sale.menu_reporting_sales",
    "d43375c1-73a6-42a2-8dbd-0f13c285824f": "sale.menu_reporting_sales"
  }
}

```

## File: data\files\sales_sample_dashboard.json

```json
{
    "version": 21,
    "sheets": [
        {
            "id": "sheet1",
            "name": "Dashboard",
            "colNumber": 7,
            "rowNumber": 86,
            "rows": {
                "5": {
                    "size": 40
                },
                "21": {
                    "size": 40
                },
                "22": {
                    "size": 29
                },
                "23": {
                    "size": 29
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
                    "size": 23
                },
                "34": {
                    "size": 43
                },
                "35": {
                    "size": 35
                },
                "36": {
                    "size": 28
                },
                "37": {
                    "size": 28
                },
                "38": {
                    "size": 28
                },
                "39": {
                    "size": 28
                },
                "40": {
                    "size": 28
                },
                "41": {
                    "size": 28
                },
                "42": {
                    "size": 28
                },
                "43": {
                    "size": 28
                },
                "44": {
                    "size": 28
                },
                "45": {
                    "size": 28
                },
                "47": {
                    "size": 40
                },
                "48": {
                    "size": 40
                },
                "49": {
                    "size": 28
                },
                "50": {
                    "size": 28
                },
                "51": {
                    "size": 28
                },
                "52": {
                    "size": 28
                },
                "53": {
                    "size": 28
                },
                "54": {
                    "size": 28
                },
                "55": {
                    "size": 28
                },
                "56": {
                    "size": 28
                },
                "57": {
                    "size": 28
                },
                "58": {
                    "size": 28
                },
                "60": {
                    "size": 40
                },
                "61": {
                    "size": 40
                },
                "62": {
                    "size": 28
                },
                "63": {
                    "size": 28
                },
                "64": {
                    "size": 28
                },
                "65": {
                    "size": 28
                },
                "66": {
                    "size": 28
                },
                "67": {
                    "size": 28
                },
                "68": {
                    "size": 28
                },
                "69": {
                    "size": 28
                },
                "70": {
                    "size": 28
                },
                "71": {
                    "size": 28
                },
                "73": {
                    "size": 40
                },
                "74": {
                    "size": 40
                },
                "75": {
                    "size": 28
                },
                "76": {
                    "size": 28
                },
                "77": {
                    "size": 28
                },
                "78": {
                    "size": 28
                },
                "79": {
                    "size": 28
                },
                "80": {
                    "size": 28
                },
                "81": {
                    "size": 28
                },
                "82": {
                    "size": 28
                },
                "83": {
                    "size": 28
                },
                "84": {
                    "size": 28
                },
                "85": {
                    "size": 28
                }
            },
            "cols": {
                "0": {
                    "size": 349
                },
                "1": {
                    "size": 95
                },
                "2": {
                    "size": 80
                },
                "3": {
                    "size": 50
                },
                "4": {
                    "size": 323
                },
                "5": {
                    "size": 100
                },
                "6": {
                    "size": 100
                }
            },
            "merges": [],
            "cells": {
                "A6": {
                    "content": "[Monthly Sales](odoo://view/{\"viewType\":\"graph\",\"action\":{\"domain\":[[\"state\",\"not in\",[\"draft\",\"cancel\",\"sent\"]]],\"context\":{\"group_by\":[\"date:month\"],\"graph_measure\":\"price_subtotal\",\"graph_mode\":\"line\",\"graph_groupbys\":[\"date:month\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
                },
                "A22": {
                    "content": "[Top Quotations](odoo://view/{\"viewType\":\"list\",\"action\":{\"domain\":[[\"state\",\"in\",[\"draft\",\"sent\"]]],\"context\":{\"group_by\":[]},\"modelName\":\"sale.order\",\"views\":[[false,\"list\"],[false,\"kanban\"],[false,\"form\"],[false,\"calendar\"],[false,\"pivot\"],[false,\"graph\"],[false,\"activity\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Quotations\"})"
                },
                "A23": {
                    "content": "=_t(\"Customer\")"
                },
                "A35": {
                    "content": "[Top Countries](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"country_id\",\"!=\",false],[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[\"country_id\"],\"pivot_measures\":[\"order_reference\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"country_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
                },
                "A36": {
                    "content": "=_t(\"Country\")"
                },
                "A48": {
                    "content": "[Top Customers](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"partner_id\",\"!=\",false],[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[\"partner_id\"],\"pivot_measures\":[\"order_reference\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"partner_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
                },
                "A49": {
                    "content": "=_t(\"Customer\")"
                },
                "A61": {
                    "content": "[Top Sales Teams](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"team_id\",\"!=\",false],[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[\"team_id\"],\"pivot_measures\":[\"order_reference\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"team_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
                },
                "A62": {
                    "content": "=_t(\"Sales Team\")"
                },
                "A74": {
                    "content": "[Top Sources](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"source_id\",\"!=\",false],[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[\"source_id\"],\"pivot_measures\":[\"order_reference\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"source_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
                },
                "A75": {
                    "content": "=_t(\"Source\")"
                },
                "B23": {
                    "content": "=_t(\"Salesperson\")"
                },
                "B36": {
                    "content": "=_t(\"Orders\")"
                },
                "B49": {
                    "content": "=_t(\"Orders\")"
                },
                "B62": {
                    "content": "=_t(\"Orders\")"
                },
                "B75": {
                    "content": "=_t(\"Orders\")"
                },
                "C23": {
                    "content": "=_t(\"Revenue\")"
                },
                "C36": {
                    "content": "=_t(\"Revenue\")"
                },
                "C49": {
                    "content": "=_t(\"Revenue\")"
                },
                "C62": {
                    "content": "=_t(\"Revenue\")"
                },
                "C75": {
                    "content": "=_t(\"Revenue\")"
                },
                "E22": {
                    "content": "[Top Sales Orders](odoo://view/{\"viewType\":\"list\",\"action\":{\"domain\":[[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[]},\"modelName\":\"sale.order\",\"views\":[[false,\"list\"],[false,\"kanban\"],[false,\"form\"],[false,\"calendar\"],[false,\"pivot\"],[false,\"graph\"],[false,\"activity\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Orders\"})"
                },
                "E23": {
                    "content": "=_t(\"Customer\")"
                },
                "E35": {
                    "content": "[Top Products](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"product_tmpl_id\",\"!=\",false],[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[\"product_id\"],\"pivot_measures\":[\"order_reference\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"product_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
                },
                "E36": {
                    "content": "=_t(\"Product\")"
                },
                "E48": {
                    "content": "[Top Categories](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"categ_id\",\"!=\",false],[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[\"categ_id\"],\"pivot_measures\":[\"order_reference\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"categ_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
                },
                "E49": {
                    "content": "=_t(\"Category\")"
                },
                "E61": {
                    "content": "[Top Salespeople](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"user_id\",\"!=\",false],[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[\"user_id\"],\"pivot_measures\":[\"order_reference\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"user_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
                },
                "E62": {
                    "content": "=_t(\"Salesperson\")"
                },
                "E74": {
                    "content": "[Top Mediums](odoo://view/{\"viewType\":\"pivot\",\"action\":{\"domain\":[[\"medium_id\",\"!=\",false],[\"state\",\"not in\",[\"draft\",\"sent\",\"cancel\"]]],\"context\":{\"group_by\":[\"medium_id\"],\"pivot_measures\":[\"order_reference\",\"price_subtotal\"],\"pivot_column_groupby\":[],\"pivot_row_groupby\":[\"medium_id\"]},\"modelName\":\"sale.report\",\"views\":[[false,\"graph\"],[false,\"pivot\"],[false,\"search\"]]},\"threshold\":0,\"name\":\"Sales Analysis\"})"
                },
                "E75": {
                    "content": "=_t(\"Medium\")"
                },
                "F23": {
                    "content": "=_t(\"Salesperson\")"
                },
                "F36": {
                    "content": "=_t(\"Orders\")"
                },
                "F49": {
                    "content": "=_t(\"Orders\")"
                },
                "F62": {
                    "content": "=_t(\"Orders\")"
                },
                "F75": {
                    "content": "=_t(\"Orders\")"
                },
                "G23": {
                    "content": "=_t(\"Revenue\")"
                },
                "G36": {
                    "content": "=_t(\"Revenue\")"
                },
                "G49": {
                    "content": "=_t(\"Revenue\")"
                },
                "G62": {
                    "content": "=_t(\"Revenue\")"
                },
                "G75": {
                    "content": "=_t(\"Revenue\")"
                }
            },
            "styles": {
                "A6": 1,
                "A22": 1,
                "A35": 1,
                "A48": 1,
                "A61": 1,
                "A74": 1,
                "E22": 1,
                "E35": 1,
                "E48": 1,
                "E61": 1,
                "E74": 1,
                "A23:B23": 2,
                "E23:F23": 2,
                "A36": 3,
                "A49": 3,
                "A62": 3,
                "A75": 3,
                "C36": 3,
                "C75": 3,
                "E36": 3,
                "E49": 3,
                "E62": 3,
                "E75": 3,
                "B36": 4,
                "B75": 4,
                "B49:C49": 4,
                "B62:C62": 4,
                "F36:G36": 4,
                "F49:G49": 4,
                "F62:G62": 4,
                "F75:G75": 4,
                "C23": 5,
                "G23": 5
            },
            "formats": {},
            "borders": {
                "A22:C22": 1,
                "A35:C35": 1,
                "A48:C48": 1,
                "A61:C61": 1,
                "A74:C74": 1,
                "A6:G6": 1,
                "E22:G22": 1,
                "E35:G35": 1,
                "E48:G48": 1,
                "E61:G61": 1,
                "E74:G74": 1,
                "B62": 2,
                "B75": 2,
                "E23:F23": 2,
                "F49": 2,
                "F62": 2,
                "F75": 2,
                "A7:G7": 2,
                "A23:B23": 3,
                "A24:B24": 4,
                "A25:B32": 5,
                "A37:C46": 5,
                "A50:C59": 5,
                "A64:C72": 5,
                "A77:C77": 5,
                "A79:C85": 5,
                "E25:G33": 5,
                "E37:G46": 5,
                "E51:G59": 5,
                "E64:G72": 5,
                "E77:G77": 5,
                "E79:G85": 5,
                "A33:B33": 6,
                "A34:C34": 7,
                "A36": 8,
                "A47:C47": 9,
                "A60:C60": 9,
                "A73:C73": 9,
                "A86:C86": 9,
                "E34:G34": 9,
                "E47:G47": 9,
                "E60:G60": 9,
                "E73:G73": 9,
                "E86:G86": 9,
                "A49": 10,
                "E36": 10,
                "A62": 11,
                "A75": 11,
                "E49": 11,
                "E62": 11,
                "E75": 11,
                "A63:C63": 12,
                "A76:C76": 12,
                "E24:G24": 12,
                "E50:G50": 12,
                "E63:G63": 12,
                "E76:G76": 12,
                "A78:C78": 13,
                "E78:G78": 13,
                "B36": 14,
                "B49": 14,
                "F36": 14,
                "C23": 15,
                "C24": 16,
                "C25:C32": 17,
                "C33": 18,
                "C36": 19,
                "C49": 20,
                "G36": 20,
                "C62": 21,
                "C75": 21,
                "G23": 21,
                "G49": 21,
                "G62": 21,
                "G75": 21,
                "D23": 22,
                "D24:D33": 23,
                "D36": 23,
                "D37:D46": 24,
                "D49:D59": 24,
                "D62:D72": 24,
                "D75:D77": 24,
                "D79:D85": 24
            },
            "conditionalFormats": [],
            "figures": [
                {
                    "id": "51823220-f22b-4359-8711-579a249c91bb",
                    "x": 0,
                    "y": 11,
                    "width": 213,
                    "height": 101,
                    "tag": "chart",
                    "data": {
                        "baselineColorDown": "#DC6965",
                        "baselineColorUp": "#00A04A",
                        "baselineMode": "percentage",
                        "title": {
                            "text": "Quotations",
                            "bold": true,
                            "color": "#434343"
                        },
                        "type": "scorecard",
                        "background": "#EFF6FF",
                        "baseline": "Data!E4",
                        "baselineDescr": "since last period",
                        "keyValue": "Data!D4",
                        "humanize": false
                    }
                },
                {
                    "id": "9a38934c-b454-4a4b-88aa-17d1b80dbf5f",
                    "x": 223,
                    "y": 11,
                    "width": 211,
                    "height": 101,
                    "tag": "chart",
                    "data": {
                        "baselineColorDown": "#DC6965",
                        "baselineColorUp": "#00A04A",
                        "baselineMode": "percentage",
                        "title": {
                            "text": "Orders",
                            "color": "#434343",
                            "bold": true
                        },
                        "type": "scorecard",
                        "background": "#EFF6FF",
                        "baseline": "Data!E5",
                        "baselineDescr": "since last period",
                        "keyValue": "Data!D5",
                        "humanize": false
                    }
                },
                {
                    "id": "67858d0e-b5ba-4a3c-bf9e-c0fceaeedf65",
                    "x": 444,
                    "y": 11,
                    "width": 218,
                    "height": 101,
                    "tag": "chart",
                    "data": {
                        "baselineColorDown": "#DC6965",
                        "baselineColorUp": "#00A04A",
                        "baselineMode": "percentage",
                        "title": {
                            "text": "Revenue",
                            "color": "#434343",
                            "bold": true
                        },
                        "type": "scorecard",
                        "background": "#FFF7ED",
                        "baseline": "Data!E7",
                        "baselineDescr": "since last period",
                        "keyValue": "Data!D7",
                        "humanize": false
                    }
                },
                {
                    "id": "d43375c1-73a6-42a2-8dbd-0f13c285824f",
                    "x": 672,
                    "y": 11,
                    "width": 213,
                    "height": 101,
                    "tag": "chart",
                    "data": {
                        "baselineColorDown": "#DC6965",
                        "baselineColorUp": "#00A04A",
                        "baselineMode": "percentage",
                        "title": {
                            "text": "Average Order",
                            "color": "#434343",
                            "bold": true
                        },
                        "type": "scorecard",
                        "background": "#FFF7ED",
                        "baseline": "Data!E8",
                        "baselineDescr": "since last period",
                        "keyValue": "Data!D8",
                        "humanize": false
                    }
                },
                {
                    "id": "3ceb14f0-2a13-4691-817e-ff15c643b2bf",
                    "x": 0,
                    "y": 156,
                    "width": 1093,
                    "height": 343,
                    "tag": "chart",
                    "data": {
                        "type": "line",
                        "dataSetsHaveTitle": false,
                        "dataSets": [
                            {
                                "dataRange": "Data!C11:C16",
                                "yAxisId": "y"
                            }
                        ],
                        "legendPosition": "none",
                        "labelRange": "Data!A11:A16",
                        "title": {},
                        "labelsAsText": true,
                        "stacked": false,
                        "aggregated": false,
                        "cumulative": true,
                        "fillArea": true
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
            "id": "eae01f9c-c461-4489-ade4-957ef2459d40",
            "name": "Data",
            "colNumber": 26,
            "rowNumber": 103,
            "rows": {},
            "cols": {},
            "merges": [],
            "cells": {
                "A1": {
                    "content": "=_t(\"KPI\")"
                },
                "A2": {
                    "content": "=_t(\"Draft quotations\")"
                },
                "A3": {
                    "content": "=_t(\"Quotations sent\")"
                },
                "A4": {
                    "content": "=_t(\"Total quotations\")"
                },
                "A5": {
                    "content": "=_t(\"Orders\")"
                },
                "A6": {
                    "content": "=_t(\"Total orders\")"
                },
                "A7": {
                    "content": "=_t(\"Revenue\")"
                },
                "A8": {
                    "content": "=_t(\"Average order amount\")"
                },
                "A11": {
                    "content": "=EDATE(TODAY(), -B11)"
                },
                "A12": {
                    "content": "=EDATE(TODAY(), -B12)"
                },
                "A13": {
                    "content": "=EDATE(TODAY(), -B13)"
                },
                "A14": {
                    "content": "=EDATE(TODAY(), -B14)"
                },
                "A15": {
                    "content": "=EDATE(TODAY(), -B15)"
                },
                "A16": {
                    "content": "=EDATE(TODAY(), -B16)"
                },
                "B1": {
                    "content": "=_t(\"Current\")"
                },
                "B2": {
                    "content": "13"
                },
                "B3": {
                    "content": "15"
                },
                "B4": {
                    "content": "189"
                },
                "B5": {
                    "content": "456"
                },
                "B6": {
                    "content": "72"
                },
                "B7": {
                    "content": "491617.3"
                },
                "B8": {
                    "content": "=IFERROR(B7/B6)"
                },
                "B11": {
                    "content": "6"
                },
                "B12": {
                    "content": "5"
                },
                "B13": {
                    "content": "4"
                },
                "B14": {
                    "content": "3"
                },
                "B15": {
                    "content": "2"
                },
                "B16": {
                    "content": "1"
                },
                "C1": {
                    "content": "=_t(\"Previous\")"
                },
                "C2": {
                    "content": "25"
                },
                "C3": {
                    "content": "25"
                },
                "C4": {
                    "content": "123"
                },
                "C5": {
                    "content": "345"
                },
                "C6": {
                    "content": "25"
                },
                "C7": {
                    "content": "350000"
                },
                "C8": {
                    "content": "=IFERROR(C7/C6)"
                },
                "C11": {
                    "content": "77913"
                },
                "C12": {
                    "content": "763749"
                },
                "C13": {
                    "content": "130466"
                },
                "C14": {
                    "content": "218483"
                },
                "C15": {
                    "content": "563073"
                },
                "C16": {
                    "content": "183723"
                },
                "D1": {
                    "content": "=_t(\"Current\")"
                },
                "D2": {
                    "content": "=B2"
                },
                "D3": {
                    "content": "15"
                },
                "D4": {
                    "content": "=B4"
                },
                "D5": {
                    "content": "=B5"
                },
                "D6": {
                    "content": "=FORMAT.LARGE.NUMBER(B6)"
                },
                "D7": {
                    "content": "=FORMAT.LARGE.NUMBER(B7)"
                },
                "D8": {
                    "content": "=FORMAT.LARGE.NUMBER(B8)"
                },
                "E1": {
                    "content": "=_t(\"Previous\")"
                },
                "E2": {
                    "content": "25"
                },
                "E3": {
                    "content": "25"
                },
                "E4": {
                    "content": "123"
                },
                "E5": {
                    "content": "345"
                },
                "E6": {
                    "content": "25"
                },
                "E7": {
                    "content": "=C7"
                },
                "E8": {
                    "content": "=C8"
                }
            },
            "styles": {
                "A1:E1": 6,
                "D2:E8": 7
            },
            "formats": {
                "A11:A16": 1,
                "C11:C16": 2,
                "B7:E8": 2
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
            "bold": true,
            "fontSize": 16
        },
        "2": {
            "fontSize": 11,
            "textColor": "#434343",
            "verticalAlign": "middle",
            "bold": true
        },
        "3": {
            "bold": true,
            "fontSize": 11,
            "textColor": "#434343"
        },
        "4": {
            "bold": true,
            "fontSize": 11,
            "textColor": "#434343",
            "align": "center"
        },
        "5": {
            "align": "center",
            "fontSize": 11,
            "textColor": "#434343",
            "verticalAlign": "middle",
            "bold": true
        },
        "6": {
            "bold": true
        },
        "7": {
            "fillColor": "#f2f2f2"
        }
    },
    "formats": {
        "1": "mmmm yyyy",
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
            "top": {
                "style": "thin",
                "color": "#CCCCCC"
            },
            "bottom": {
                "style": "thin",
                "color": "#FFFFFF"
            }
        },
        "4": {
            "top": {
                "style": "thin",
                "color": "#FFFFFF"
            },
            "bottom": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "left": {
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
            },
            "bottom": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "left": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "right": {
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
                "style": "thin",
                "color": "#FFFFFF"
            },
            "left": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "right": {
                "style": "thick",
                "color": "#FFFFFF"
            }
        },
        "7": {
            "top": {
                "style": "thin",
                "color": "#FFFFFF"
            }
        },
        "8": {
            "top": {
                "style": "thin",
                "color": "#CCCCCC"
            },
            "bottom": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "left": {
                "style": "thin",
                "color": "#FFFFFF"
            }
        },
        "9": {
            "top": {
                "style": "thick",
                "color": "#FFFFFF"
            }
        },
        "10": {
            "top": {
                "style": "thin",
                "color": "#CCCCCC"
            },
            "bottom": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "left": {
                "style": "thick",
                "color": "#FFFFFF"
            }
        },
        "11": {
            "top": {
                "style": "thin",
                "color": "#CCCCCC"
            },
            "left": {
                "style": "thick",
                "color": "#FFFFFF"
            }
        },
        "12": {
            "bottom": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "left": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "right": {
                "style": "thick",
                "color": "#FFFFFF"
            }
        },
        "13": {
            "top": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "bottom": {
                "style": "thick",
                "color": "#FFFFFF"
            }
        },
        "14": {
            "top": {
                "style": "thin",
                "color": "#CCCCCC"
            },
            "bottom": {
                "style": "thick",
                "color": "#FFFFFF"
            }
        },
        "15": {
            "top": {
                "style": "thin",
                "color": "#CCCCCC"
            },
            "bottom": {
                "style": "thin",
                "color": "#FFFFFF"
            },
            "right": {
                "style": "thin",
                "color": "#FFFFFF"
            }
        },
        "16": {
            "top": {
                "style": "thin",
                "color": "#FFFFFF"
            },
            "bottom": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "left": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "right": {
                "style": "thin",
                "color": "#FFFFFF"
            }
        },
        "17": {
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
            },
            "right": {
                "style": "thin",
                "color": "#FFFFFF"
            }
        },
        "18": {
            "top": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "bottom": {
                "style": "thin",
                "color": "#FFFFFF"
            },
            "left": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "right": {
                "style": "thin",
                "color": "#FFFFFF"
            }
        },
        "19": {
            "top": {
                "style": "thin",
                "color": "#CCCCCC"
            },
            "bottom": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "right": {
                "style": "thin",
                "color": "#FFFFFF"
            }
        },
        "20": {
            "top": {
                "style": "thin",
                "color": "#CCCCCC"
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
        "21": {
            "top": {
                "style": "thin",
                "color": "#CCCCCC"
            },
            "right": {
                "style": "thick",
                "color": "#FFFFFF"
            }
        },
        "22": {
            "left": {
                "style": "thin",
                "color": "#FFFFFF"
            }
        },
        "23": {
            "left": {
                "style": "thin",
                "color": "#FFFFFF"
            },
            "right": {
                "style": "thick",
                "color": "#FFFFFF"
            }
        },
        "24": {
            "left": {
                "style": "thick",
                "color": "#FFFFFF"
            },
            "right": {
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
    "pivotNextId": 13,
    "customTableStyles": {},
    "odooVersion": 12,
    "globalFilters": [],
    "lists": {},
    "listNextId": 3
}

```

