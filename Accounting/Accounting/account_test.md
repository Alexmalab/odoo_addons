# Odoo Module: account_test

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2011 CCI Connect asbl (http://www.cciconnect.be) All Rights Reserved.
#                       Philmer <philmer@cciconnect.be>

{
    'name': 'Accounting Consistency Tests',
    'version': '1.0',
    'category': 'Accounting/Accounting',
    'description': """
Asserts on accounting.
======================
With this module you can manually check consistencies and inconsistencies of accounting module from menu Reporting/Accounting/Accounting Tests.

You can write a query in order to create Consistency Test and you will get the result of the test 
in PDF format which can be accessed by Menu Reporting -> Accounting Tests, then select the test 
and print the report from Print button in header area.
""",
    'depends': ['account'],
    'data': [
        'security/ir.model.access.csv',
        'views/accounting_assert_test_views.xml',
        'report/accounting_assert_test_reports.xml',
        'data/accounting_assert_test_data.xml',
        'report/report_account_test_templates.xml',
    ],
    'active': False,
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: data\accounting_assert_test_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="accounting.assert.test" id="account_test_01">
        <field name="sequence">1</field>
        <field name="name">Test 1: General balance</field>
        <field name="desc">Check the balance: Debit sum = Credit sum</field>
        <field name="code_exec"><![CDATA[sql="""SELECT
sum(debit)-sum(credit) as balance
FROM  account_move_line 
"""
cr.execute(sql)
result=[]
res= cr.dictfetchall()
if res[0]['balance']!=0.0 and res[0]['balance'] is not None:
  result.append(_('* The difference of the balance is: '))
  result.append(res)
]]></field>
    </record>

    <record model="accounting.assert.test" id="account_test_03">
        <field name="sequence">3</field>
        <field name="name">Test 3: Movement lines</field>
        <field name="desc">Check if movement lines are balanced and have the same date and period</field>
        <field name="code_exec"><![CDATA[order_columns=['am_date','ml_date','am.date','ml.date','am.id']
sql="""SELECT
  am.id as move_id,
  sum(debit)-sum(credit) as balance,
  am.date,
  ml.date,
  am.date as am_date,
  ml.date as ml_date
FROM account_move am, account_move_line ml
WHERE
  ml.move_id = am.id
GROUP BY am.name, am.id, am.state, am.date, ml.date,am.date, ml.date,am.date, ml.date
HAVING abs(sum(ml.debit-ml.credit)) <> 0 or am.date!=ml.date or (am.date!=ml.date)
"""
cr.execute(sql)
res = cr.dictfetchall()
if res:
    res.insert(0,_('* The test failed for these movement lines:'))
result = res

]]></field>
    </record>

<!-- TODO: rewrite test since the model of reconciliation has changed -->
<!--     <record model="accounting.assert.test" id="account_test_04">
        <field name="sequence">4</field>
        <field name="name">Test 4: Totally reconciled journal items</field>
        <field name="desc">Check if the totally reconciled journal items are balanced</field>
        <field name="code_exec"><![CDATA[res = []
cr.execute("SELECT distinct reconcile_id from account_move_line where reconcile_id is not null")
rec_ids = cr.dictfetchall()
for record in rec_ids :
  cr.execute("SELECT distinct r.name,r.id from account_journal j,account_period p, account_move_reconcile r,account_move m, account_move_line ml where m.journal_id=j.id and m.date=p.id and ml.reconcile_id=%s and ml.move_id=m.id and ml.reconcile_id=r.id group by r.id,r.name having sum(ml.debit)-sum(ml.credit)<>0", (record['reconcile_id'],))
  reconcile_ids=cr.dictfetchall()
  if reconcile_ids:
    res.append(', '.join(["Reconcile name: %(name)s, id=%(id)s " % r for r in reconcile_ids]))
result = res
if result:
    result.insert(0,_('* The test failed for these reconciled items(id/name):'))
]]></field>
    </record> -->

    <record model="accounting.assert.test" id="account_test_05">
        <field name="sequence">5</field>
        <field name="name">Test 5.1 : Payable and Receivable accountant lines of reconciled invoices</field>
        <field name="desc">Check that reconciled invoice for Sales/Purchases has reconciled entries for Payable and Receivable Accounts</field>
        <field name="code_exec"><![CDATA[res = []
cr.execute("SELECT distinct inv.number,inv.id from account_invoice inv, account_move m, account_move_line ml, account_account a where m.id=ml.move_id and ml.account_id=a.id and a.internal_type in ('receivable','payable') and inv.move_id=m.id and ml.reconciled is true;")
records= cr.dictfetchall()
rec = [r['id'] for r in records]
res = reconciled_inv()
invoices = set(rec).difference(set(res))
result = [rec for rec in records if rec['id'] in invoices]
if result:
    result.insert(0,_('* Invoices that need to be checked: '))
]]></field>
    </record>

    <record model="accounting.assert.test" id="account_test_05_2">
        <field name="sequence">6</field>
        <field name="name">Test 5.2 : Reconcilied invoices and Payable/Receivable accounts</field>
        <field name="desc">Check that reconciled account moves, that define Payable and Receivable accounts, are belonging to reconciled invoices</field>
        <field name="code_exec"><![CDATA[res = reconciled_inv()
result=[]
if res:
    cr.execute("SELECT distinct inv.number,inv.id from account_invoice inv, account_move_line ml, account_account a, account_move m where m.id=ml.move_id and inv.move_id=m.id and inv.id=inv.move_id and ml.reconciled is false and a.internal_type in ('receivable','payable') and ml.account_id=a.id and inv.id in %s",(tuple(res),))
    records = cr.dictfetchall()
    result = [rec for rec in records]
    if result:
        result.insert(0,_('* Invoices that need to be checked: '))
]]></field>
    </record>

    <record model="accounting.assert.test" id="account_test_06">
        <field name="sequence">7</field>
        <field name="name">Test 6 : Invoices status</field>
        <field name="desc">Check that paid/reconciled invoices are not in 'Open' state</field>
        <field name="code_exec"><![CDATA[
from odoo import _
res = []
column_order = ['number','id','name','state']
if reconciled_inv():
  cr.execute("select inv.name,inv.state,inv.id,inv.number from account_invoice inv where inv.state!='paid' and id in %s", (tuple(reconciled_inv()),))
  res = cr.dictfetchall()
result = res
if result:
    result.insert(0,_('* Invoices that need to be checked: '))
]]></field>
    </record>

    <record model="accounting.assert.test" id="account_test_07">
        <field name="sequence">8</field>
        <field name="name">Test 7 : Closing balance on bank statements</field>
        <field name="desc">Check on bank statement that the Closing Balance = Starting Balance + sum of statement lines</field>
        <field name="code_exec"><![CDATA[column_order = ['name','difference']
cr.execute("SELECT s.balance_start+sum(m.amount)-s.balance_end_real as difference, s.name from account_bank_statement s inner join account_bank_statement_line m on m.statement_id=s.id group by s.id, s.balance_start, s.balance_end_real,s.name having abs(s.balance_start+sum(m.amount)-s.balance_end_real) > 0.000000001;")
result = cr.dictfetchall()
if result:
    result.insert(0,_('* Unbalanced bank statement that need to be checked: '))
]]></field>
    </record>
    <!-- TODO account.period has been removed -->
    <!-- <record model="accounting.assert.test" id="account_test_08">
        <field name="sequence">9</field>
        <field name="name">Test 8 : Accounts and partners on account moves</field>
        <field name="desc">Check that general accounts and partners on account moves are active</field>
        <field name="code_exec"><![CDATA[column_order=['partner_name','partner_active','account_name','move_line_id','period']
res = []
cr.execute("SELECT l.id as move_line_id,a.name as account_name,a.code as account_code,r.name as partner_name,r.active as partner_active,p.name as period from account_period p,res_partner r, account_account a,account_move_line l where l.account_id=a.id and l.partner_id=r.id and (not r.active or not a.active) and l.period_id=p.id")
res = cr.dictfetchall()
result = res
if result:
  result.insert(0,_('* Here is the list of inactive partners and movement lines that are not correct: '))
]]></field>
    </record> -->
</odoo>

```

## File: models\accounting_assert_test.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

CODE_EXEC_DEFAULT = '''\
res = []
cr.execute("select id, code from account_journal")
for record in cr.dictfetchall():
    res.append(record['code'])
result = res
'''


class AccountingAssertTest(models.Model):
    _name = "accounting.assert.test"
    _description = 'Accounting Assert Test'
    _order = "sequence"

    name = fields.Char(string='Test Name', required=True, index=True, translate=True)
    desc = fields.Text(string='Test Description', index=True, translate=True)
    code_exec = fields.Text(string='Python code', required=True, default=CODE_EXEC_DEFAULT)
    active = fields.Boolean(default=True)
    sequence = fields.Integer(default=10)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import accounting_assert_test

```

## File: report\accounting_assert_test_reports.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <report
            id="account_assert_test_report" 
            model="accounting.assert.test" 
            string="Accounting Tests"
            report_type="qweb-pdf"
            name="account_test.report_accounttest" 
            file="account_test.report_accounttest" 
        />
</odoo>

```

## File: report\report_account_test.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
from odoo import api, models, _
from odoo.tools.safe_eval import safe_eval
#
# Use period and Journal for selection or resources
#


class ReportAssertAccount(models.AbstractModel):
    _name = 'report.account_test.report_accounttest'
    _description = 'Account Test Report'

    @api.model
    def execute_code(self, code_exec):
        def reconciled_inv():
            """
            returns the list of invoices that are set as reconciled = True
            """
            return self.env['account.move'].search([('reconciled', '=', True)]).ids

        def order_columns(item, cols=None):
            """
            This function is used to display a dictionary as a string, with its columns in the order chosen.

            :param item: dict
            :param cols: list of field names
            :returns: a list of tuples (fieldname: value) in a similar way that would dict.items() do except that the
                returned values are following the order given by cols
            :rtype: [(key, value)]
            """
            if cols is None:
                cols = list(item)
            return [(col, item.get(col)) for col in cols if col in item]

        localdict = {
            'cr': self.env.cr,
            'uid': self.env.uid,
            'reconciled_inv': reconciled_inv,  # specific function used in different tests
            'result': None,  # used to store the result of the test
            'column_order': None,  # used to choose the display order of columns (in case you are returning a list of dict)
            '_': _,
        }
        safe_eval(code_exec, localdict, mode="exec", nocopy=True)
        result = localdict['result']
        column_order = localdict.get('column_order', None)

        if not isinstance(result, (tuple, list, set)):
            result = [result]
        if not result:
            result = [_('The test was passed successfully')]
        else:
            def _format(item):
                if isinstance(item, dict):
                    return ', '.join(["%s: %s" % (tup[0], tup[1]) for tup in order_columns(item, column_order)])
                else:
                    return item
            result = [_format(rec) for rec in result]

        return result

    @api.model
    def _get_report_values(self, docids, data=None):
        report = self.env['ir.actions.report']._get_report_from_name('account_test.report_accounttest')
        records = self.env['accounting.assert.test'].browse(self.ids)
        return {
            'doc_ids': self._ids,
            'doc_model': report.model,
            'docs': records,
            'data': data,
            'execute_code': self.execute_code,
            'datetime': datetime
        }

```

## File: report\report_account_test_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="report_accounttest">
    <t t-call="web.html_container">
        <t t-call="web.internal_layout">
            <div class="page">
                <h2>Accounting tests on <span t-esc="datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')"/></h2>
                <div t-foreach="docs" t-as="o">
                    <p>
                        <strong>Name:</strong> <span t-field="o.name"/><br/>
                        <strong>Description:</strong> <span t-field="o.desc"/>
                    </p>
                    <p t-foreach="execute_code(o.code_exec)" t-as="test_result">
                        <span t-esc="test_result"/>
                    </p>
                </div>
            </div>
        </t>
    </t>
</template>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import report_account_test

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
"access_accounting_assert_test","accounting.assert.test","model_accounting_assert_test",base.group_system,1,0,0,1
"access_accounting_assert_test_manager","accounting.assert.test","model_accounting_assert_test",account.group_account_manager,1,0,0,0

```

## File: views\accounting_assert_test_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record model="ir.ui.view" id="account_assert_tree">
            <field name="name">accounting.assert.test.tree</field>
            <field name="model">accounting.assert.test</field>
            <field name="arch" type="xml">
                <tree string="Tests">
                    <field name="sequence"/>
                    <field name="name"/>
                    <field name="desc"/>
                </tree>
            </field>
        </record>

        <record model="ir.ui.view" id="account_assert_form">
            <field name="name">accounting.assert.test.form</field>
            <field name="model">accounting.assert.test</field>
            <field name="arch" type="xml">
                <form string="Tests">
                    <sheet>
                        <group>
                            <group>
                                <field name="name"/>
                                <field name="sequence"/>
                            </group>
                            <group>
                                <field name="active"/>
                            </group>
                        </group>
                        <notebook>
                            <page string="Description">
                                <field name="desc" nolabel="1"/>
                            </page>
                            <page string="Expression">
                                <group string="Python Code">
                                    <field colspan="4" name="code_exec" nolabel="1"/>
                                </group>
                                <group string="Code Help">
                                    <pre>
Code should always set a variable named `result` with the result of your test, that can be a list or
a dictionary. If `result` is an empty list, it means that the test was successful. Otherwise it will
try to translate and print what is inside `result`.

If the result of your test is a dictionary, you can set a variable named `column_order` to choose in
what order you want to print `result`'s content.

Should you need them, you can also use the following variables into your code:
    * cr: cursor to the database
    * uid: ID of the current user

In any ways, the code must be legal python statements with correct indentation (if needed).

Example: 
    sql = '''SELECT id, name, ref, date
             FROM account_move_line 
             WHERE account_id IN (SELECT id FROM account_account WHERE type = 'view')
          '''
    cr.execute(sql)
    result = cr.dictfetchall()
                                    </pre>
                                </group>
                            </page>
                        </notebook>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="accounting_assert_test_view_search" model="ir.ui.view">
            <field name="name">accounting.assert.test.view.search</field>
            <field name="model">accounting.assert.test</field>
            <field name="arch" type="xml">
                <search string="Search Account Test">
                    <field string="Name" name="name"/>
                    <field string="Description" name="desc"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                </search>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_accounting_assert">
            <field name="name">Accounting Tests</field>
            <field name="res_model">accounting.assert.test</field>
            <field name="view_mode">tree,form</field>
            <field name="search_view_id" ref="accounting_assert_test_view_search"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new accounting test
              </p>
            </field>
        </record>

        <menuitem name="Accounting Tests" parent="account.menu_finance_reports" id="menu_action_license" action="action_accounting_assert" sequence="50" groups="base.group_no_one"/>

</odoo>

```

