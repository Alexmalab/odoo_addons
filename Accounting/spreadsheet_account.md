# Odoo Module: spreadsheet_account

Category: Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': "Spreadsheet Accounting Formulas",
    'version': '1.0',
    'category': 'Accounting',
    'summary': 'Spreadsheet Accounting formulas',
    'description': 'Spreadsheet Accounting formulas',
    'depends': ['spreadsheet', 'account'],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
    'assets': {
        'spreadsheet.o_spreadsheet': [
            (
                'after',
                'spreadsheet/static/src/o_spreadsheet/o_spreadsheet.js',
                'spreadsheet_account/static/src/**/*.js'
            ),
        ],
        'web.assets_unit_tests': [
            'spreadsheet_account/static/tests/**/*',
        ],
    }
}

```

## File: models\account.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import date
import calendar
from dateutil.relativedelta import relativedelta

from odoo import models, api, _
from odoo.osv import expression
from odoo.tools import date_utils


class AccountMove(models.Model):
    _inherit = "account.account"

    @api.model
    def _get_date_period_boundaries(self, date_period, company):
        period_type = date_period["range_type"]
        year = date_period.get("year")
        month = date_period.get("month")
        quarter = date_period.get("quarter")
        day = date_period.get("day")
        if period_type == "year":
            fiscal_day = company.fiscalyear_last_day
            fiscal_month = int(company.fiscalyear_last_month)
            if not (fiscal_day == 31 and fiscal_month == 12):
                year += 1
            max_day = calendar.monthrange(year, fiscal_month)[1]
            current = date(year, fiscal_month, min(fiscal_day, max_day))
            start, end = date_utils.get_fiscal_year(current, fiscal_day, fiscal_month)
        elif period_type == "month":
            start = date(year, month, 1)
            end = start + relativedelta(months=1, days=-1)
        elif period_type == "quarter":
            first_month = quarter * 3 - 2
            start = date(year, first_month, 1)
            end = start + relativedelta(months=3, days=-1)
        elif period_type == "day":
            fiscal_day = company.fiscalyear_last_day
            fiscal_month = int(company.fiscalyear_last_month)
            end = date(year, month, day)
            start, _ = date_utils.get_fiscal_year(end, fiscal_day, fiscal_month)
        return start, end

    def _build_spreadsheet_formula_domain(self, formula_params, default_accounts=False):
        codes = [code for code in formula_params["codes"] if code]

        default_domain = expression.FALSE_DOMAIN
        if not codes:
            if not default_accounts:
                return default_domain
            default_domain = [('account_type', 'in', ['liability_payable', 'asset_receivable'])]

        company_id = formula_params["company_id"] or self.env.company.id
        company = self.env["res.company"].browse(company_id)
        start, end = self._get_date_period_boundaries(
            formula_params["date_range"], company
        )
        balance_domain = [
            ("account_id.include_initial_balance", "=", True),
            ("date", "<=", end),
        ]
        pnl_domain = [
            ("account_id.include_initial_balance", "=", False),
            ("date", ">=", start),
            ("date", "<=", end),
        ]
        # It is more optimized to (like) search for code directly in account.account than in account_move_line
        code_domain = expression.OR(
            [
                ("code", "=like", f"{code}%"),
            ]
            for code in codes
        )
        account_domain = expression.OR([code_domain, default_domain])
        account_ids = self.env["account.account"].with_company(company_id).search(account_domain).ids
        code_domain = [("account_id", "in", account_ids)]
        period_domain = expression.OR([balance_domain, pnl_domain])
        domain = expression.AND([code_domain, period_domain, [("company_id", "=", company_id)]])
        if formula_params["include_unposted"]:
            domain = expression.AND(
                [domain, [("move_id.state", "!=", "cancel")]]
            )
        else:
            domain = expression.AND(
                [domain, [("move_id.state", "=", "posted")]]
            )
        partner_ids = [int(partner_id) for partner_id in formula_params.get('partner_ids', []) if partner_id]
        if partner_ids:
            domain = expression.AND(
                [domain, [("partner_id", "in", partner_ids)]]
            )
        return domain

    @api.model
    def spreadsheet_move_line_action(self, args):
        domain = self._build_spreadsheet_formula_domain(args, default_accounts=True)
        return {
            "type": "ir.actions.act_window",
            "res_model": "account.move.line",
            "view_mode": "list",
            "views": [[False, "list"]],
            "target": "current",
            "domain": domain,
            "name": _("Cell Audit"),
        }

    @api.model
    def spreadsheet_fetch_debit_credit(self, args_list):
        """Fetch data for ODOO.CREDIT, ODOO.DEBIT and ODOO.BALANCE formulas
        The input list looks like this:
        [{
            date_range: {
                range_type: "year"
                year: int
            },
            company_id: int
            codes: str[]
            include_unposted: bool
        }]
        """
        results = []
        for args in args_list:
            company_id = args["company_id"] or self.env.company.id
            domain = self._build_spreadsheet_formula_domain(args)
            MoveLines = self.env["account.move.line"].with_company(company_id)
            [(debit, credit)] = MoveLines._read_group(domain, aggregates=['debit:sum', 'credit:sum'])
            results.append({'debit': debit or 0, 'credit': credit or 0})

        return results

    @api.model
    def spreadsheet_fetch_residual_amount(self, args_list):
        """Fetch data for ODOO.RESUDUAL formulas
        The input list looks like this:
        [{
            date_range: {
                range_type: "year"
                year: int
            },
            company_id: int
            codes: str[]
            include_unposted: bool
        }]
        """
        results = []
        for args in args_list:
            company_id = args["company_id"] or self.env.company.id
            domain = self._build_spreadsheet_formula_domain(args, default_accounts=True)
            MoveLines = self.env["account.move.line"].with_company(company_id)
            [(amount_residual,)] = MoveLines._read_group(domain, aggregates=['amount_residual:sum'])
            results.append({'amount_residual': amount_residual or 0})

        return results

    @api.model
    def spreadsheet_fetch_partner_balance(self, args_list):
        """Fetch data for ODOO.PARTNER.BALANCE formulas
        The input list looks like this:
        [{
            date_range: {
                range_type: "year"
                year: int
            },
            company_id: int
            codes: str[]
            include_unposted: bool
            partner_ids: int[]
        }]
        """
        results = []
        for args in args_list:
            partner_ids = [partner_id for partner_id in args.get('partner_ids', []) if partner_id]
            if not partner_ids:
                results.append({'balance': 0})
                continue

            company_id = args["company_id"] or self.env.company.id
            domain = self._build_spreadsheet_formula_domain(args, default_accounts=True)
            MoveLines = self.env["account.move.line"].with_company(company_id)
            [(balance,)] = MoveLines._read_group(domain, aggregates=['balance:sum'])
            results.append({'balance': balance or 0})

        return results

    @api.model
    def get_account_group(self, account_types):
        data = self._read_group(
            [
                *self._check_company_domain(self.env.company),
                ("account_type", "in", account_types),
            ],
            ['account_type'],
            ['code:array_agg'],
        )
        mapped = dict(data)
        return [mapped.get(account_type, []) for account_type in account_types]

```

## File: models\res_company.py

```python
from odoo import models, api, fields

from odoo.tools import date_utils


class ResCompany(models.Model):
    _inherit = "res.company"

    @api.model
    def get_fiscal_dates(self, payload):
        companies = self.env["res.company"].browse(
            data["company_id"] or self.env.company.id for data in payload
        )
        existing_companies = companies.exists()
        # prefetch both fields
        existing_companies.fetch(["fiscalyear_last_day", "fiscalyear_last_month"])
        results = []

        for data, company in zip(payload, companies):
            if company not in existing_companies:
                results.append(False)
                continue
            start, end = date_utils.get_fiscal_year(
                fields.Date.to_date(data["date"]),
                day=company.fiscalyear_last_day,
                month=int(company.fiscalyear_last_month),
            )
            results.append({"start": start, "end": end})
        return results

```

## File: models\__init__.py

```python
from . import account
from . import res_company

```

## File: static\src\accounting_functions.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { sprintf } from "@web/core/utils/strings";

import * as spreadsheet from "@odoo/o-spreadsheet";
import { EvaluationError } from "@odoo/o-spreadsheet";
const { functionRegistry } = spreadsheet.registries;
const { arg, toBoolean, toString, toNumber, toJsDate } = spreadsheet.helpers;

const QuarterRegexp = /^q([1-4])\/(\d{4})$/i;
const MonthRegexp = /^0?([1-9]|1[0-2])\/(\d{4})$/i;

/**
 * @typedef {Object} YearDateRange
 * @property {"year"} rangeType
 * @property {number} year
 */

/**
 * @typedef {Object} QuarterDateRange
 * @property {"quarter"} rangeType
 * @property {number} year
 * @property {number} quarter
 */

/**
 * @typedef {Object} MonthDateRange
 * @property {"month"} rangeType
 * @property {number} year
 * @property {number} month
 */

/**
 * @typedef {Object} DayDateRange
 * @property {"day"} rangeType
 * @property {number} year
 * @property {number} month
 * @property {number} day
 */

/**
 * @typedef {YearDateRange | QuarterDateRange | MonthDateRange | DayDateRange} DateRange
 */

/**
 * @param {object | undefined} dateRange
 * @returns {QuarterDateRange | undefined}
 */
function parseAccountingQuarter(dateRange) {
    const found = toString(dateRange?.value).trim().match(QuarterRegexp);
    return found
        ? {
              rangeType: "quarter",
              year: Number(found[2]),
              quarter: Number(found[1]),
          }
        : undefined;
}

/**
 * @param {object | undefined} dateRange
 * @returns {MonthDateRange | undefined}
 */
function parseAccountingMonth(dateRange, locale) {
    if (
        typeof dateRange?.value === "number" &&
        dateRange.format?.includes("m") &&
        !dateRange.format?.includes("d")
    ) {
        const date = toJsDate(dateRange.value, locale);
        return {
            rangeType: "month",
            year: date.getFullYear(),
            month: date.getMonth() + 1,
        };
    }
    const found = toString(dateRange?.value).trim().match(MonthRegexp);
    return found
        ? {
              rangeType: "month",
              year: Number(found[2]),
              month: Number(found[1]),
          }
        : undefined;
}

/**
 * @param {object | undefined} dateRange
 * @returns {YearDateRange | undefined}
 */
function parseAccountingYear(dateRange, locale) {
    const dateNumber = toNumber(dateRange?.value, locale);
    // This allows a bit of flexibility for the user if they were to input a
    // numeric value instead of a year.
    // Users won't need to fetch accounting info for year 3000 before a long time
    // And the numeric value 3000 corresponds to 18th march 1908, so it's not an
    //issue to prevent them from fetching accounting data prior to that date.
    if (dateNumber < 3000) {
        return { rangeType: "year", year: dateNumber };
    }
    return undefined;
}

/**
 * @param {object | undefined} dateRange
 * @returns {DayDateRange}
 */
function parseAccountingDay(dateRange, locale) {
    const dateNumber = toNumber(dateRange?.value, locale);
    return {
        rangeType: "day",
        year: functionRegistry.get("YEAR").compute.bind({ locale })(dateNumber),
        month: functionRegistry.get("MONTH").compute.bind({ locale })(dateNumber),
        day: functionRegistry.get("DAY").compute.bind({ locale })(dateNumber),
    };
}

/**
 * @param {object | undefined} dateRange
 * @returns {DateRange}
 */
export function parseAccountingDate(dateRange, locale) {
    try {
        return (
            parseAccountingQuarter(dateRange) ||
            parseAccountingMonth(dateRange, locale) ||
            parseAccountingYear(dateRange, locale) ||
            parseAccountingDay(dateRange, locale)
        );
    } catch {
        throw new EvaluationError(
            sprintf(
                _t(
                    `'%s' is not a valid period. Supported formats are "21/12/2022", "Q1/2022", "12/2022", and "2022".`
                ),
                dateRange?.value
            )
        );
    }
}

const YEAR_OFFSET_ARG = arg("offset (number, default=0)", _t("Offset applied to the years."))
const COMPANY_ARG = arg("company_id (number, optional)", _t("The company to target (Advanced)."))
const POSTED_ARG = arg(
    "include_unposted (boolean, default=FALSE)",
    _t("Set to TRUE to include unposted entries.")
)

const ODOO_FIN_ARGS = () => [
    arg("account_codes (string)", _t("The prefix of the accounts.")),
    arg(
        "date_range (string, date)",
        _t(`The date range. Supported formats are "21/12/2022", "Q1/2022", "12/2022", and "2022".`)
    ),
    YEAR_OFFSET_ARG,
    COMPANY_ARG,
    POSTED_ARG,
];

const ODOO_RESIDUAL_ARGS = () => [
    arg(
        "account_codes (string, optional)",
        _t("The prefix of the accounts. If none provided, all receivable and payable accounts will be used.")
    ),
    arg(
        "date_range (string, date, optional)",
        _t(`The date range. Supported formats are "21/12/2022", "Q1/2022", "12/2022", and "2022".`)
    ),
    YEAR_OFFSET_ARG,
    COMPANY_ARG,
    POSTED_ARG,
];

const ODOO_PARTNER_BALANCE_ARGS = () => {
    const partner_arg = arg("partner_ids (string)", _t("The partner ids (separated by a comma)."));
    return [partner_arg, ...ODOO_RESIDUAL_ARGS()];
}

functionRegistry.add("ODOO.CREDIT", {
    description: _t("Get the total credit for the specified account(s) and period."),
    args: ODOO_FIN_ARGS(),
    category: "Odoo",
    returns: ["NUMBER"],
    compute: function (
        accountCodes,
        dateRange,
        offset = { value: 0 },
        companyId = { value: null },
        includeUnposted = { value: false }
    ) {
        const _accountCodes = toString(accountCodes)
            .split(",")
            .map((code) => code.trim())
            .sort();
        const _offset = toNumber(offset, this.locale);
        const _dateRange = parseAccountingDate(dateRange, this.locale);
        const _companyId = companyId?.value;
        const _includeUnposted = toBoolean(includeUnposted);
        return {
            value: this.getters.getAccountPrefixCredit(
                _accountCodes,
                _dateRange,
                _offset,
                _companyId,
                _includeUnposted
            ),
            format: this.getters.getCompanyCurrencyFormat(_companyId) || "#,##0.00",
        };
    },
});

functionRegistry.add("ODOO.DEBIT", {
    description: _t("Get the total debit for the specified account(s) and period."),
    args: ODOO_FIN_ARGS(),
    category: "Odoo",
    returns: ["NUMBER"],
    compute: function (
        accountCodes,
        dateRange,
        offset = { value: 0 },
        companyId = { value: null },
        includeUnposted = { value: false }
    ) {
        const _accountCodes = toString(accountCodes)
            .split(",")
            .map((code) => code.trim())
            .sort();
        const _offset = toNumber(offset, this.locale);
        const _dateRange = parseAccountingDate(dateRange, this.locale);
        const _companyId = companyId?.value;
        const _includeUnposted = toBoolean(includeUnposted);
        return {
            value: this.getters.getAccountPrefixDebit(
                _accountCodes,
                _dateRange,
                _offset,
                _companyId,
                _includeUnposted
            ),
            format: this.getters.getCompanyCurrencyFormat(_companyId) || "#,##0.00",
        };
    },
});

functionRegistry.add("ODOO.BALANCE", {
    description: _t("Get the total balance for the specified account(s) and period."),
    args: ODOO_FIN_ARGS(),
    category: "Odoo",
    returns: ["NUMBER"],
    compute: function (
        accountCodes,
        dateRange,
        offset = { value: 0 },
        companyId = { value: null },
        includeUnposted = { value: false }
    ) {
        const _accountCodes = toString(accountCodes)
            .split(",")
            .map((code) => code.trim())
            .sort();
        const _offset = toNumber(offset, this.locale);
        const _dateRange = parseAccountingDate(dateRange, this.locale);
        const _companyId = companyId?.value;
        const _includeUnposted = toBoolean(includeUnposted);
        const value =
            this.getters.getAccountPrefixDebit(
                _accountCodes,
                _dateRange,
                _offset,
                _companyId,
                _includeUnposted
            ) -
            this.getters.getAccountPrefixCredit(
                _accountCodes,
                _dateRange,
                _offset,
                _companyId,
                _includeUnposted
            );
        return { value, format: this.getters.getCompanyCurrencyFormat(_companyId) || "#,##0.00" };
    },
});

functionRegistry.add("ODOO.FISCALYEAR.START", {
    description: _t("Returns the starting date of the fiscal year encompassing the provided date."),
    args: [
        arg("day (date)", _t("The day from which to extract the fiscal year start.")),
        arg("company_id (number, optional)", _t("The company.")),
    ],
    category: "Odoo",
    returns: ["NUMBER"],
    compute: function (date, companyId = { value: null }) {
        const startDate = this.getters.getFiscalStartDate(
            toJsDate(date, this.locale),
            companyId.value === null ? null : toNumber(companyId, this.locale)
        );
        return {
            value: toNumber(startDate, this.locale),
            format: this.locale.dateFormat,
        };
    },
});

functionRegistry.add("ODOO.FISCALYEAR.END", {
    description: _t("Returns the ending date of the fiscal year encompassing the provided date."),
    args: [
        arg("day (date)", _t("The day from which to extract the fiscal year end.")),
        arg("company_id (number, optional)", _t("The company.")),
    ],
    category: "Odoo",
    returns: ["NUMBER"],
    compute: function (date, companyId = { value: null }) {
        const endDate = this.getters.getFiscalEndDate(
            toJsDate(date, this.locale),
            companyId.value === null ? null : toNumber(companyId, this.locale)
        );
        return {
            value: toNumber(endDate, this.locale),
            format: this.locale.dateFormat,
        };
    },
});

const ACCOUNT_TYPES = [
    "asset_receivable",
    "asset_cash",
    "asset_current",
    "asset_non_current",
    "asset_prepayments",
    "asset_fixed",
    "liability_payable",
    "liability_credit_card",
    "liability_current",
    "liability_non_current",
    "equity",
    "equity_unaffected",
    "income",
    "income_other",
    "expense",
    "expense_depreciation",
    "expense_direct_cost",
    "off_balance",
];

functionRegistry.add("ODOO.ACCOUNT.GROUP", {
    description: _t("Returns the account codes of a given group."),
    args: [
        arg(
            "type (string)",
            _t("The technical account type (possible values are: %s).", ACCOUNT_TYPES.join(", "))
        ),
    ],
    category: "Odoo",
    returns: ["NUMBER"],
    compute: function (accountType) {
        const accountTypes = this.getters.getAccountGroupCodes(toString(accountType));
        return accountTypes.join(",");
    },
});

functionRegistry.add("ODOO.RESIDUAL", {
    description: _t("Return the residual amount for the specified account(s) and period"),
    args: ODOO_RESIDUAL_ARGS(),
    category: "Odoo",
    returns: ["NUMBER"],
    compute: function (
        accountCodes,
        dateRange,
        offset = { value: 0 },
        companyId = { value: null },
        includeUnposted = { value: false }
    ) {
        const _accountCodes = toString(accountCodes)
            .split(",")
            .map((code) => code.trim())
            .sort();
        const _offset = toNumber(offset, this.locale);
        if ( !dateRange?.value ) {
            dateRange = { value: new Date().getFullYear() }
        }
        const _dateRange = parseAccountingDate(dateRange, this.locale);
        const _companyId = toNumber(companyId, this.locale);
        const _includeUnposted = toBoolean(includeUnposted);
        return {
            value: this.getters.getAccountResidual(
                _accountCodes,
                _dateRange,
                _offset,
                _companyId,
                _includeUnposted
            ),
            format: this.getters.getCompanyCurrencyFormat(_companyId) || "#,##0.00",
        };
    },
})

functionRegistry.add("ODOO.PARTNER.BALANCE", {
    description: _t("Return the partner balance for the specified account(s) and period"),
    args: ODOO_PARTNER_BALANCE_ARGS(),
    category: "Odoo",
    returns: ["NUMBER"],
    compute: function (
        partnerIds,
        accountCodes,
        dateRange,
        offset = { value: 0 },
        companyId = { value: null },
        includeUnposted = { value: false }
    ) {
        const _partnerIds = toString(partnerIds)
            .split(",")
            .map((partnerId) => toNumber(partnerId, this.locale))
            .sort();
        const _accountCodes = toString(accountCodes)
            .split(",")
            .map((code) => code.trim())
            .sort();
        const _offset = toNumber(offset, this.locale);

        if ( !dateRange?.value ) {
            dateRange = { value: new Date().getFullYear() }
        }
        const _dateRange = parseAccountingDate(dateRange, this.locale);
        const _companyId = toNumber(companyId, this.locale);
        const _includeUnposted = toBoolean(includeUnposted);
        return {
            value: this.getters.getAccountPartnerData(
                _accountCodes,
                _dateRange,
                _offset,
                _companyId,
                _includeUnposted,
                _partnerIds
            ),
            format: this.getters.getCompanyCurrencyFormat(_companyId) || "#,##0.00",
        };
    },
})

```

## File: static\src\account_group_auto_complete.js

```javascript
import { _t } from "@web/core/l10n/translation";

import { registries, tokenColors, helpers } from "@odoo/o-spreadsheet";

const { insertTokenAfterLeftParenthesis } = helpers;

// copy-pasted list of options from the `account_type` selection field.
const ACCOUNT_TYPES = [
    ["asset_receivable", _t("Receivable")],
    ["asset_cash", _t("Bank and Cash")],
    ["asset_current", _t("Current Assets")],
    ["asset_non_current", _t("Non-current Assets")],
    ["asset_prepayments", _t("Prepayments")],
    ["asset_fixed", _t("Fixed Assets")],
    ["liability_payable", _t("Payable")],
    ["liability_credit_card", _t("Credit Card")],
    ["liability_current", _t("Current Liabilities")],
    ["liability_non_current", _t("Non-current Liabilities")],
    ["equity", _t("Equity")],
    ["equity_unaffected", _t("Current Year Earnings")],
    ["income", _t("Income")],
    ["income_other", _t("Other Income")],
    ["expense", _t("Expenses")],
    ["expense_depreciation", _t("Depreciation")],
    ["expense_direct_cost", _t("Cost of Revenue")],
    ["off_balance", _t("Off-Balance Sheet")],
];

registries.autoCompleteProviders.add("account_group_types", {
    sequence: 50,
    autoSelectFirstProposal: true,
    getProposals(tokenAtCursor) {
        const functionContext = tokenAtCursor.functionContext;
        if (
            functionContext?.parent.toUpperCase() === "ODOO.ACCOUNT.GROUP" &&
            functionContext.argPosition === 0
        ) {
            return ACCOUNT_TYPES.map(([technicalName, displayName]) => {
                const text = `"${technicalName}"`;
                return {
                    text,
                    description: displayName,
                    htmlContent: [{ value: text, color: tokenColors.STRING }],
                    fuzzySearchKey: technicalName + displayName,
                };
            });
        }
        return;
    },
    selectProposal: insertTokenAfterLeftParenthesis,
});

```

## File: static\src\index.js

```javascript
/** @odoo-module */

import { _t } from "@web/core/l10n/translation";
import * as spreadsheet from "@odoo/o-spreadsheet";
import { AccountingPlugin } from "./plugins/accounting_plugin";
import { getFirstAccountFunction, getNumberOfAccountFormulas } from "./utils";
import { parseAccountingDate } from "./accounting_functions";
import { camelToSnakeObject } from "@spreadsheet/helpers/helpers";

const { cellMenuRegistry, featurePluginRegistry } = spreadsheet.registries;
const { astToFormula } = spreadsheet;
const { isEvaluationError, toString, toBoolean } = spreadsheet.helpers;

featurePluginRegistry.add("odooAccountingAggregates", AccountingPlugin);

cellMenuRegistry.add("move_lines_see_records", {
    name: _t("See records"),
    sequence: 176,
    async execute(env) {
        const position = env.model.getters.getActivePosition();
        const sheetId = position.sheetId;
        const cell = env.model.getters.getCell(position);
        const func = getFirstAccountFunction(cell.compiledFormula.tokens);
        let codes, partner_ids = "";
        let date_range, offset, companyId, includeUnposted = false;
        const parsed_args = func.args.map(astToFormula).map(
            (arg) => env.model.getters.evaluateFormulaResult(sheetId, arg)
        );
        if ( func.functionName === "ODOO.PARTNER.BALANCE" ) {
            [partner_ids, codes, date_range, offset, companyId, includeUnposted] = parsed_args;
        } else {
            [codes, date_range, offset, companyId, includeUnposted] = parsed_args;
        }
        if ( codes?.value && !isEvaluationError(codes.value) ) {
            codes = toString(codes?.value).split(",").map((code) => code.trim());
        } else {
            codes = [];
        }
        const locale = env.model.getters.getLocale();
        let dateRange;
        if ( date_range?.value && !isEvaluationError(date_range.value) ) {
            dateRange = parseAccountingDate(date_range, locale);
        } else {
            if ( ["ODOO.PARTNER.BALANCE", "ODOO.RESIDUAL"].includes(func.functionName) ) {
                dateRange = parseAccountingDate({ value: new Date().getFullYear() }, locale);
            }
        }
        offset = parseInt(offset?.value) || 0;
        dateRange.year += offset || 0;
        companyId = parseInt(companyId?.value) || null;
        try {
            includeUnposted = toBoolean(includeUnposted.value);
        } catch {
            includeUnposted = false;
        }
        const partnerIds = toString(partner_ids).split(",").map((code) => code.trim());

        let param;
        if ( func.functionName === "ODOO.PARTNER.BALANCE" ) {
            param = [camelToSnakeObject({ dateRange, companyId, codes, includeUnposted, partnerIds })]
        } else {
            param = [camelToSnakeObject({ dateRange, companyId, codes, includeUnposted })]
        }
        const action = await env.services.orm.call(
            "account.account",
            "spreadsheet_move_line_action",
            param
        );
        await env.services.action.doAction(action);
    },
    isVisible: (env) => {
        const position = env.model.getters.getActivePosition();
        const evaluatedCell = env.model.getters.getEvaluatedCell(position);
        const cell = env.model.getters.getCell(position);
        return (
            !isEvaluationError(evaluatedCell.value) &&
            evaluatedCell.value !== "" &&
            cell &&
            cell.isFormula &&
            getNumberOfAccountFormulas(cell.compiledFormula.tokens) === 1
        );
    },
    icon: "o-spreadsheet-Icon.SEE_RECORDS",
});

```

## File: static\src\utils.js

```javascript
/** @odoo-module **/
// @ts-check

import { helpers } from "@odoo/o-spreadsheet";

const { getFunctionsFromTokens } = helpers;

/**
 * @typedef {import("@odoo/o-spreadsheet").Token} Token
 * @typedef  {import("@spreadsheet/helpers/odoo_functions_helpers").OdooFunctionDescription} OdooFunctionDescription
 */

/**
 * @param {Token[]} tokens
 * @returns {number}
 */
export function getNumberOfAccountFormulas(tokens) {
    return getFunctionsFromTokens(tokens, ["ODOO.BALANCE", "ODOO.CREDIT", "ODOO.DEBIT", "ODOO.RESIDUAL", "ODOO.PARTNER.BALANCE"]).length;
}

/**
 * Get the first Account function description of the given formula.
 *
 * @param {Token[]} tokens
 * @returns {OdooFunctionDescription | undefined}
 */
export function getFirstAccountFunction(tokens) {
    return getFunctionsFromTokens(tokens, ["ODOO.BALANCE", "ODOO.CREDIT", "ODOO.DEBIT", "ODOO.RESIDUAL", "ODOO.PARTNER.BALANCE"])[0];
}

```

## File: static\src\plugins\accounting_plugin.js

```javascript
/** @odoo-module */
// @ts-check

import { EvaluationError } from "@odoo/o-spreadsheet";
import { OdooUIPlugin } from "@spreadsheet/plugins";
import { _t } from "@web/core/l10n/translation";
import { deepCopy } from "@web/core/utils/objects";
import { camelToSnakeObject, toServerDateString } from "@spreadsheet/helpers/helpers";

/**
 * @typedef {import("../accounting_functions").DateRange} DateRange
 */

export class AccountingPlugin extends OdooUIPlugin {
    static getters = /** @type {const} */ ([
        "getAccountPrefixCredit",
        "getAccountPrefixDebit",
        "getAccountGroupCodes",
        "getFiscalStartDate",
        "getFiscalEndDate",
        "getAccountResidual",
        "getAccountPartnerData",
    ]);
    constructor(config) {
        super(config);
        /** @type {import("@spreadsheet/data_sources/server_data").ServerData} */
        this._serverData = config.custom.odooDataProvider?.serverData;
    }

    get serverData() {
        if (!this._serverData) {
            throw new Error(
                "'serverData' is not defined, please make sure a 'OdooDataProvider' instance is provided to the model."
            );
        }
        return this._serverData;
    }

    // -------------------------------------------------------------------------
    // Getters
    // -------------------------------------------------------------------------

    /**
     * Gets the total balance for given account code prefix
     * @param {string[]} codes prefixes of the accounts' codes
     * @param {DateRange} dateRange start date of the period to look
     * @param {number} offset end  date of the period to look
     * @param {number | null} companyId specific company to target
     * @param {boolean} includeUnposted wether or not select unposted entries
     * @returns {number}
     */
    getAccountPrefixCredit(codes, dateRange, offset, companyId, includeUnposted) {
        const data = this._fetchAccountData(codes, dateRange, offset, companyId, includeUnposted);
        return data.credit;
    }

    /**
     * Gets the total balance for a given account code prefix
     * @param {string[]} codes prefixes of the accounts codes
     * @param {DateRange} dateRange start date of the period to look
     * @param {number} offset end  date of the period to look
     * @param {number | null} companyId specific company to target
     * @param {boolean} includeUnposted wether or not select unposted entries
     * @returns {number}
     */
    getAccountPrefixDebit(codes, dateRange, offset, companyId, includeUnposted) {
        const data = this._fetchAccountData(codes, dateRange, offset, companyId, includeUnposted);
        return data.debit;
    }

    /**
     * @param {Date} date Date included in the fiscal year
     * @param {number | null} companyId specific company to target
     * @returns {string | undefined}
     */
    getFiscalStartDate(date, companyId) {
        return this._fetchCompanyData(date, companyId).start;
    }

    /**
     * @param {Date} date Date included in the fiscal year
     * @param {number | undefined} companyId specific company to target
     * @returns {string | undefined}
     */
    getFiscalEndDate(date, companyId) {
        return this._fetchCompanyData(date, companyId).end;
    }

    /**
     * @param {string} accountType
     * @returns {string[]}
     */
    getAccountGroupCodes(accountType) {
        return this.serverData.batch.get("account.account", "get_account_group", accountType);
    }

    /**
     * Fetch the account information (credit/debit) for a given account code
     * @private
     * @param {string[]} codes prefix of the accounts' codes
     * @param {DateRange} dateRange start date of the period to look
     * @param {number} offset end  date of the period to look
     * @param {number | null} companyId specific companyId to target
     * @param {boolean} includeUnposted wether or not select unposted entries
     * @returns {{ debit: number, credit: number }}
     */
    _fetchAccountData(codes, dateRange, offset, companyId, includeUnposted) {
        dateRange = deepCopy(dateRange);
        dateRange.year += offset;
        // Excel dates start at 1899-12-30, we should not support date ranges
        // that do not cover dates prior to it.
        // Unfortunately, this check needs to be done right before the server
        // call as a date to low (year <= 1) can raise an error server side.
        if (dateRange.year < 1900) {
            throw new EvaluationError(_t("%s is not a valid year.", dateRange.year));
        }
        return this.serverData.batch.get(
            "account.account",
            "spreadsheet_fetch_debit_credit",
            camelToSnakeObject({ dateRange, codes, companyId, includeUnposted })
        );
    }

    /**
     * Fetch the start and end date of the fiscal year enclosing a given date
     * Defaults on the current user company if not provided
     * @private
     * @param {Date} date
     * @param {number | null} companyId
     * @returns {{start: string, end: string}}
     */
    _fetchCompanyData(date, companyId) {
        const result = this.serverData.batch.get("res.company", "get_fiscal_dates", {
            date: toServerDateString(date),
            company_id: companyId,
        });
        if (result === false) {
            throw new EvaluationError(_t("The company fiscal year could not be found."));
        }
        return result;
    }

    /**
     * Gets the residual amount for given account code prefixes over a given period
     * @param {string[]} codes prefixes of the accounts codes
     * @param {DateRange} dateRange start date of the period to look
     * @param {number} offset year offset of the period to search
     * @param {number} companyId specific company to target
     * @param {boolean} includeUnposted whether or not select unposted entries
     * @returns {number | undefined}
     */
    getAccountResidual(codes, dateRange, offset, companyId, includeUnposted) {
        dateRange = deepCopy(dateRange);
        dateRange.year += offset;
        // Excel dates start at 1899-12-30, we should not support date ranges
        // that do not cover dates prior to it.
        // Unfortunately, this check needs to be done right before the server
        // call as a date to low (year <= 1) can raise an error server side.
        if (dateRange.year < 1900) {
            throw new EvaluationError(_t("%s is not a valid year.", dateRange.year));
        }
        const result = this.serverData.batch.get(
            "account.account",
            "spreadsheet_fetch_residual_amount",
            camelToSnakeObject({ codes, dateRange, companyId, includeUnposted })
        );
        if (result === false) {
            throw new EvaluationError(_t("The residual amount for given accounts could not be computed."));
        }
        return result.amount_residual;
    }

    /**
     * Fetch the account information for a given account code and partner
     * @private
     * @param {string[]} codes prefix of the accounts' codes
     * @param {DateRange} dateRange start date of the period to look
     * @param {number} offset year offset of the period to look
     * @param {number | null} companyId specific companyId to target
     * @param {boolean} includeUnposted wether or not select unposted entries
     * @param {number[]} partnerIds ids of the partners
     * @returns {number | undefined}
     */
    getAccountPartnerData(codes, dateRange, offset, companyId, includeUnposted, partnerIds) {
        dateRange = deepCopy(dateRange);
        dateRange.year += offset;
        // Excel dates start at 1899-12-30, we should not support date ranges
        // that do not cover dates prior to it.
        // Unfortunately, this check needs to be done right before the server
        // call as a date to low (year <= 1) can raise an error server side.
        if (dateRange.year < 1900) {
            throw new EvaluationError(_t("%s is not a valid year.", dateRange.year));
        }
        const result = this.serverData.batch.get(
            "account.account",
            "spreadsheet_fetch_partner_balance",
            camelToSnakeObject({ dateRange, codes, companyId, includeUnposted, partnerIds })
        );
        if (result === false) {
            throw new EvaluationError(_t("The balance for given partners could not be computed."));
        }
        return result.balance;
    }
}

```

