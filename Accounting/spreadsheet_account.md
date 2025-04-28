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
        'web.qunit_suite_tests': [
            'spreadsheet_account/static/tests/**/*.js',
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

    def _build_spreadsheet_formula_domain(self, formula_params):
        codes = [code for code in formula_params["codes"] if code]
        if not codes:
            return expression.FALSE_DOMAIN
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
        account_ids = self.env["account.account"].with_company(company_id).search(code_domain).ids
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
        return domain

    @api.model
    def spreadsheet_move_line_action(self, args):
        domain = self._build_spreadsheet_formula_domain(args)
        return {
            "type": "ir.actions.act_window",
            "res_model": "account.move.line",
            "view_mode": "list",
            "views": [[False, "list"]],
            "target": "current",
            "domain": domain,
            "name": _("Journal items for account prefix %s", ", ".join(args["codes"])),
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
            # remove this when _search always returns a Query object
            if expression.is_false(self.env["account.move.line"], domain):
                results.append({"credit": 0, "debit": 0})
                continue
            MoveLines = self.env["account.move.line"].with_company(company_id)
            query = MoveLines._search(domain)
            query_str, params = query.select(
                "SUM(debit) AS debit", "SUM(credit) AS credit"
            )
            self.env.cr.execute(query_str, params)
            line_values = self.env.cr.dictfetchone()
            results.append(
                {
                    "credit": line_values["credit"] or 0,
                    "debit": line_values["debit"] or 0,
                }
            )

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

## File: static\src\accounting_datasource.js

```javascript
/** @odoo-module */
import { camelToSnakeObject, toServerDateString } from "@spreadsheet/helpers/helpers";
import { _t } from "@web/core/l10n/translation";
import { sprintf } from "@web/core/utils/strings";
import { deepCopy } from "@web/core/utils/objects";

import { ServerData } from "@spreadsheet/data_sources/server_data";

/**
 * @typedef {import("./accounting_functions").DateRange} DateRange
 */

export class AccountingDataSource {
    constructor(services) {
        this.serverData = new ServerData(services.orm, {
            whenDataIsFetched: () => services.notify(),
        });
    }

    /**
     * Gets the total credit for a given account code prefix
     * @param {string[]} codes prefixes of the accounts codes
     * @param {DateRange} dateRange start date of the period to look
     * @param {number} offset end date of the period to look
     * @param {number} companyId specific company to target
     * @param {boolean} includeUnposted wether or not select unposted entries
     * @returns {number | undefined}
     */
    getCredit(codes, dateRange, offset, companyId, includeUnposted) {
        const data = this._fetchAccountData(codes, dateRange, offset, companyId, includeUnposted);
        return data.credit;
    }

    /**
     * Gets the total debit for a given account code prefix
     * @param {string[]} codes prefixes of the accounts codes
     * @param {DateRange} dateRange start date of the period to look
     * @param {number} offset end  date of the period to look
     * @param {number} companyId specific company to target
     * @param {boolean} includeUnposted wether or not select unposted entries
     * @returns {number | undefined}
     */
    getDebit(codes, dateRange, offset, companyId, includeUnposted) {
        const data = this._fetchAccountData(codes, dateRange, offset, companyId, includeUnposted);
        return data.debit;
    }

    /**
     * @param {Date} date
     * @param {number | null} companyId
     * @returns {string}
     */
    getFiscalStartDate(date, companyId) {
        return this._fetchCompanyData(date, companyId).start;
    }

    /**
     * @param {Date} date
     * @param {number | null} companyId
     * @returns {string}
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
            throw new Error(sprintf(_t("%s is not a valid year."), dateRange.year));
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
            throw new Error(_t("The company fiscal year could not be found."));
        }
        return result;
    }
}

```

## File: static\src\accounting_functions.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { sprintf } from "@web/core/utils/strings";

import * as spreadsheet from "@odoo/o-spreadsheet";
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
        throw new Error(
            sprintf(
                _t(
                    `'%s' is not a valid period. Supported formats are "21/12/2022", "Q1/2022", "12/2022", and "2022".`
                ),
                dateRange?.value
            )
        );
    }
}

const ODOO_FIN_ARGS = () => [
    arg("account_codes (string)", _t("The prefix of the accounts.")),
    arg(
        "date_range (string, date)",
        _t(`The date range. Supported formats are "21/12/2022", "Q1/2022", "12/2022", and "2022".`)
    ),
    arg("offset (number, default=0)", _t("Year offset applied to date_range.")),
    arg("company_id (number, optional)", _t("The company to target (Advanced).")),
    arg(
        "include_unposted (boolean, default=FALSE)",
        _t("Set to TRUE to include unposted entries.")
    ),
];

functionRegistry.add("ODOO.CREDIT", {
    description: _t("Get the total credit for the specified account(s) and period."),
    args: ODOO_FIN_ARGS(),
    category: "Odoo",
    returns: ["NUMBER"],
    computeValueAndFormat: function (
        accountCodes,
        dateRange,
        offset = { value: 0 },
        companyId = { value: null },
        includeUnposted = { value: false }
    ) {
        accountCodes = toString(accountCodes?.value)
            .split(",")
            .map((code) => code.trim())
            .sort();
        offset = toNumber(offset.value, this.locale);
        dateRange = parseAccountingDate(dateRange, this.locale);
        includeUnposted = toBoolean(includeUnposted.value);
        const value = this.getters.getAccountPrefixCredit(
            accountCodes,
            dateRange,
            offset,
            companyId.value,
            includeUnposted
        );
        const format = this.getters.getCompanyCurrencyFormat(companyId.value) || "#,##0.00";
        return { value, format };
    },
});

functionRegistry.add("ODOO.DEBIT", {
    description: _t("Get the total debit for the specified account(s) and period."),
    args: ODOO_FIN_ARGS(),
    category: "Odoo",
    returns: ["NUMBER"],
    computeValueAndFormat: function (
        accountCodes,
        dateRange,
        offset = { value: 0 },
        companyId = { value: null },
        includeUnposted = { value: false }
    ) {
        accountCodes = toString(accountCodes?.value)
            .split(",")
            .map((code) => code.trim())
            .sort();
        offset = toNumber(offset.value, this.locale);
        dateRange = parseAccountingDate(dateRange, this.locale);
        includeUnposted = toBoolean(includeUnposted.value);
        const value = this.getters.getAccountPrefixDebit(
            accountCodes,
            dateRange,
            offset,
            companyId.value,
            includeUnposted
        );
        const format = this.getters.getCompanyCurrencyFormat(companyId.value) || "#,##0.00";
        return { value, format };
    },
});

functionRegistry.add("ODOO.BALANCE", {
    description: _t("Get the total balance for the specified account(s) and period."),
    args: ODOO_FIN_ARGS(),
    category: "Odoo",
    returns: ["NUMBER"],
    computeValueAndFormat: function (
        accountCodes,
        dateRange,
        offset = { value: 0 },
        companyId = { value: null },
        includeUnposted = { value: false }
    ) {
        accountCodes = toString(accountCodes?.value)
            .split(",")
            .map((code) => code.trim())
            .sort();
        offset = toNumber(offset.value, this.locale);
        dateRange = parseAccountingDate(dateRange, this.locale);
        includeUnposted = toBoolean(includeUnposted.value);
        const value =
            this.getters.getAccountPrefixDebit(
                accountCodes,
                dateRange,
                offset,
                companyId.value,
                includeUnposted
            ) -
            this.getters.getAccountPrefixCredit(
                accountCodes,
                dateRange,
                offset,
                companyId.value,
                includeUnposted
            );
        const format = this.getters.getCompanyCurrencyFormat(companyId.value) || "#,##0.00";
        return { value, format };
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
    computeFormat: function () {
        return this.locale.dateFormat;
    },
    compute: function (date, companyId = null) {
        const startDate = this.getters.getFiscalStartDate(
            toJsDate(date, this.locale),
            companyId === null ? null : toNumber(companyId, this.locale)
        );
        return toNumber(startDate, this.locale);
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
    computeFormat: function () {
        return this.locale.dateFormat;
    },
    compute: function (date, companyId = null) {
        const endDate = this.getters.getFiscalEndDate(
            toJsDate(date, this.locale),
            companyId === null ? null : toNumber(companyId, this.locale)
        );
        return toNumber(endDate, this.locale);
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
const { toString, toBoolean } = spreadsheet.helpers;

featurePluginRegistry.add("odooAccountingAggregates", AccountingPlugin);

cellMenuRegistry.add("move_lines_see_records", {
    name: _t("See records"),
    sequence: 176,
    async execute(env) {
        const position = env.model.getters.getActivePosition();
        const sheetId = position.sheetId;
        const cell = env.model.getters.getCell(position);
        const { args } = getFirstAccountFunction(cell.compiledFormula.tokens);
        let [codes, date_range, offset, companyId, includeUnposted] = args
            .map(astToFormula)
            .map((arg) => env.model.getters.evaluateFormulaResult(sheetId, arg));
        codes = toString(codes?.value).split(",");
        const locale = env.model.getters.getLocale();
        const dateRange = parseAccountingDate(date_range, locale);
        offset = parseInt(offset?.value) || 0;
        dateRange.year += offset || 0;
        companyId = parseInt(companyId?.value) || null;
        try {
            includeUnposted = toBoolean(includeUnposted.value);
        } catch {
            includeUnposted = false;
        }

        const action = await env.services.orm.call(
            "account.account",
            "spreadsheet_move_line_action",
            [camelToSnakeObject({ dateRange, companyId, codes, includeUnposted })]
        );
        await env.services.action.doAction(action);
    },
    isVisible: (env) => {
        const position = env.model.getters.getActivePosition();
        const evaluatedCell = env.model.getters.getEvaluatedCell(position);
        const cell = env.model.getters.getCell(position);
        return (
            !evaluatedCell.error &&
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
import { getOdooFunctions } from "@spreadsheet/helpers/odoo_functions_helpers";

/**
 * @typedef {import("@spreadsheet/helpers/odoo_functions_helpers").Token} Token
 * @typedef  {import("@spreadsheet/helpers/odoo_functions_helpers").OdooFunctionDescription} OdooFunctionDescription
 */

/**
 * @param {Token[]} tokens
 * @returns {number}
 */
export function getNumberOfAccountFormulas(tokens) {
    return getOdooFunctions(tokens, ["ODOO.BALANCE", "ODOO.CREDIT", "ODOO.DEBIT"]).length;
}

/**
 * Get the first Account function description of the given formula.
 *
 * @param {Token[]} tokens
 * @returns {OdooFunctionDescription | undefined}
 */
export function getFirstAccountFunction(tokens) {
    return getOdooFunctions(tokens, ["ODOO.BALANCE", "ODOO.CREDIT", "ODOO.DEBIT"])[0];
}

```

## File: static\src\plugins\accounting_plugin.js

```javascript
/** @odoo-module */

import * as spreadsheet from "@odoo/o-spreadsheet";
import { AccountingDataSource } from "../accounting_datasource";
const DATA_SOURCE_ID = "ACCOUNTING_AGGREGATES";

/**
 * @typedef {import("../accounting_functions").DateRange} DateRange
 */

export class AccountingPlugin extends spreadsheet.UIPlugin {
    constructor(config) {
        super(config);
        this.dataSources = config.custom.dataSources;
        if (this.dataSources) {
            this.dataSources.add(DATA_SOURCE_ID, AccountingDataSource);
        }
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
        return (
            this.dataSources &&
            this.dataSources
                .get(DATA_SOURCE_ID)
                .getCredit(codes, dateRange, offset, companyId, includeUnposted)
        );
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
        return (
            this.dataSources &&
            this.dataSources
                .get(DATA_SOURCE_ID)
                .getDebit(codes, dateRange, offset, companyId, includeUnposted)
        );
    }

    /**
     * @param {Date} date Date included in the fiscal year
     * @param {number | null} companyId specific company to target
     * @returns {string | undefined}
     */
    getFiscalStartDate(date, companyId) {
        return (
            this.dataSources &&
            this.dataSources.get(DATA_SOURCE_ID).getFiscalStartDate(date, companyId)
        );
    }

    /**
     * @param {Date} date Date included in the fiscal year
     * @param {number | undefined} companyId specific company to target
     * @returns {string | undefined}
     */
    getFiscalEndDate(date, companyId) {
        return (
            this.dataSources &&
            this.dataSources.get(DATA_SOURCE_ID).getFiscalEndDate(date, companyId)
        );
    }

    /**
     * @param {string} accountType
     * @returns {string[]}
     */
    getAccountGroupCodes(accountType) {
        return (
            this.dataSources &&
            this.dataSources.get(DATA_SOURCE_ID).getAccountGroupCodes(accountType)
        );
    }
}

AccountingPlugin.getters = [
    "getAccountPrefixCredit",
    "getAccountPrefixDebit",
    "getAccountGroupCodes",
    "getFiscalStartDate",
    "getFiscalEndDate",
];

```

