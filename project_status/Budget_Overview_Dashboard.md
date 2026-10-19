# Logbook — Budget Overview Dashboard

This design specification is based on [GitHub issue #10: Design the budget
overview and explain each number it displays](https://github.com/KomeOn/Python_CA/issues/10).
It defines the dashboard content, proposed meanings for its figures, and
screen interactions. It does not implement the screen or calculations.

## Purpose

Give the user a clear, actionable view of recorded income, expenses, savings,
investments, budget position, recurring payments, and the estimated impact of
a planned purchase.

Every figure must reconcile to the records, applicable budgets, or explicitly
labeled projection behind it. Users must be able to open those supporting
records from a chart, metric, insight, or payment.

## Dashboard layout

```text
Logbook                                      [Notifications] [Add record]
[Overview] [Transactions] [Planning] [Investments] [Purchase checker]

Period: [Month v] [Previous] [Current period] [Next]
View:   [Actual] [Planned] [Projected]

Monthly summary
  Income          Expenses         Savings         Investments
  Net surplus / deficit            Budget used     Budget remaining
  Compared with previous month     [View supporting records]

Budget distribution
  [Donut: spending / savings / investments / remaining net cash]
  [Income vs Expenses | Savings | Investments] [Explain this chart v]

Activity over time
  [Monthly grouped bars: income | expenses | savings | investments]
  [Explain this chart v]

Recurring payments
  Name       Expected amount   Frequency   Next date   Status
  ...        ...               ...         ...         ...
  Expected recurring payments this period: [amount]

Purchase impact projector
  Amount [      ] Category [      v] Proposed date [      ]
  [ ] Essential     [Check impact]

Alerts and insights
  [Reason and amount] [Review related records]
```

On smaller screens, place the summary first and stack chart cards and lists
vertically. Keep the selected period visible. Charts and metrics remain
keyboard-accessible and have text equivalents.

## Reporting controls

- Support **monthly**, **quarterly**, **half-yearly (6 months)**, and
  **yearly** periods.
- Display the selected period's exact start and end dates.
- Use the user's configured currency, date format, and time zone.
- Apply consistent period boundaries to every dashboard component.
- Allow the user to distinguish **actual**, **planned**, and **projected**
  information. Never present a scheduled or forecast item as a completed
  transaction.
- Comparisons use the immediately preceding period of the same duration,
  where comparable records exist. Label the comparison period explicitly.

## Monthly financial summary

Show each amount separately; do not combine unlike measures into an
unexplained total.

| Figure | Proposed meaning | Open on selection |
|---|---|---|
| **Total income** | Sum of recorded income transactions in the selected period. Exclude transfers between the user's own accounts. | Transactions filtered to income and period |
| **Total expenses** | Sum of recorded expense transactions in the selected period, adjusted for linked refunds/reversals according to the shared ledger rules. Exclude internal transfers, investment purchases, and savings allocations unless the product explicitly classifies an item otherwise. | Transactions filtered to expenses and period |
| **Total savings** | Actual savings contributions recorded in the period, separately from planned contributions. A transfer/allocation is not new income or a second asset. | Transactions and goal records filtered to contributions |
| **Total investments** | Actual investment contributions/purchases recorded in the period, not portfolio market value or investment return. Show market value separately in the investment area if available. | Investment transactions filtered to period |
| **Net surplus / deficit** | Proposed cash-flow measure: recorded income minus recorded expenses for the period, excluding internal transfers. Show savings and investment contributions separately and indicate whether this net figure is before those allocations. | Underlying income and expense transactions |
| **Budget utilization** | Actual eligible spending divided by the applicable configured budget limit, as a percentage. Show “Not configured” if no applicable budget exists; define how multiple category limits relate to an overall budget before implementation. | Relevant budget and included expenses |
| **Remaining budget** | Configured budget limit minus actual eligible spending to date in the selected period. It may be negative when overspent. | Relevant budget and included expenses |
| **Previous-period change** | Difference and percentage change for each comparable measure, with its comparison period. Avoid a percentage when the prior value is zero or incomparable. | Current and comparison records |

These are proposed definitions to make the issue's requested figures
understandable. Confirm them against the shared financial model before
implementation, especially the treatment of refunds, savings, investments,
and budgets. The same definitions must be used in charts, reports, and the
purchase projector.

### Monthly insight

Summarize whether the selected period appears **improving**, **stable**, or
**deteriorating** only when there is enough comparable data and a defined
comparison rule. State the main observed factors, such as a recorded change
in spending, income, or savings. Link each claim to the records supporting
it. If data is insufficient, say so rather than inferring a trend.

Use text and accessible indicators in addition to color for positive,
negative, and attention states.

## Budget distribution circular chart

The issue requests circular graphs for income, spending, savings,
investments, and surplus or deficit. These values are not automatically
separate slices of one whole: savings and investments can be movements of
income, and a deficit is negative. Showing them together as an ordinary pie
could double-count or misrepresent the money.

**Proposed safe presentation:** Use a donut only for a clearly defined,
reconciled allocation of the selected-period income:

- Spending
- Savings contributions
- Investment contributions
- Remaining net cash (surplus), when non-negative

Show a deficit as a separate negative indicator rather than a positive donut
slice. Do not render the donut as a valid allocation if categories overlap,
the totals do not reconcile, or required data is incomplete. In those cases,
show separate labeled amounts and explain why a distribution cannot be
calculated.

Each circular chart must show:

- Selected period, total basis, category amounts, and percentages.
- A legend with distinguishable labels and colors; do not rely on color
  alone.
- A collapsible explanation covering what is measured, how it is calculated,
  the period, the key takeaway, and any warning or data limitation.
- A drill-down to the contributing records, preserving date, category,
  account, and transaction-type filters as applicable.
- A useful empty state if no qualifying transactions exist.

**Decision required:** Confirm whether the proposed allocation is the
intended meaning of “budget distribution.” If not, choose a chart that
doesn't imply overlapping categories make up one whole.

## Monthly activity histogram

Show a grouped bar chart with one group per month and separate series for
recorded **income**, **expenses**, **savings contributions**, and
**investment contributions**.

- Let users compare multiple months within the selected period or a clearly
  identified comparison range.
- Label each series and show amounts through visible labels or an accessible
  tooltip.
- Use a consistent zero baseline; represent negative or reversed values
  distinctly and explain them.
- Do not combine investment contributions with market-value changes.
- Selecting a bar opens the transactions or financial records that contributed
  to that value.
- Show a meaningful empty state when there is no data.
- Include the same collapsible explanation fields as the circular chart.

## Recurring and subscribed payments

Show user-recorded subscriptions, bills, loan payments, and other scheduled
payments with:

- Name and category.
- Expected amount and currency.
- Frequency and next scheduled date.
- Payment status, if known.
- Monthly equivalent for non-monthly schedules.
- Expected recurring-expense total for the selected month.

Highlight upcoming payments and flag overdue or failed payments only when
their status is known. If history supports it, identify amount changes and
link to the comparison. Selecting an item opens its recurring-payment record
or the linked transaction.

Do **not** mark a payment as paid just because its scheduled date has passed.
An unknown status must remain unknown until the user or a reliable data source
confirms it.

## Purchase impact projector

The projector is a dashboard feature using shared accounts, transactions,
budgets, and recurring records. It must not create a separate balance or
automatically record a purchase.

### Inputs

- Purchase amount.
- Purchase category.
- Proposed purchase date.
- Optional essential/discretionary designation.

### Information used

- Current available balance from the selected account or accounts.
- Remaining budget for the category and period, when configured.
- Expected income and recorded expenses applicable to the period.
- Known upcoming recurring payments.
- Projected balance after the proposed purchase.
- Estimated impact on monthly surplus or deficit.
- Whether the purchase exceeds a category or overall budget.

### Result

Show before-and-after estimated position, budget fit, projected available
funds, and the main reasons for the result. Explain trade-offs such as reduced
savings capacity or insufficient funds for upcoming commitments. Link to the
accounts, budgets, transactions, and recurring records used.

Present two distinct questions:

1. **Can available funds cover it after known commitments?**
2. **Does it fit the configured budget?**

A purchase can pass one and fail the other. Label all future values as
estimates, communicate incomplete data, and do not guarantee future income or
expenses. Do not create an actual transaction unless the user separately
confirms it through a clearly defined transaction workflow.

**Decision required:** Issue #10 does not prescribe the exact forecast
horizon, how expected income is weighted, how multiple accounts are selected,
or how missing/stale balances affect the result. Define those rules before
implementation.

## Traceability and interaction behavior

- Selecting a metric, chart segment, or bar opens a filtered transaction list
  or the relevant budget/recurring record.
- Preserve relevant filters: reporting period, category, account, and
  transaction type.
- Show the number of contributing transactions where useful.
- Label manually entered values, estimates, projected values, and data derived
  from incomplete records.
- For a figure derived from a budget or schedule rather than a single
  transaction, open that budget or recurring record.
- When no matching records exist, explain that clearly and provide a
  relevant action such as adding a transaction or configuring a budget.
- Updating a transaction or budget refreshes all affected dashboard values.
- Handle refunds, reversals, and corrections consistently with the underlying
  ledger; do not silently change historical interpretations.

## States and accessibility

- **Loading:** Keep the selected reporting period visible and indicate that
  figures are being loaded or recalculated.
- **Empty:** Explain what data is missing and link to the relevant setup or
  add-record flow. Do not use zero to imply a known balance when none exists.
- **Incomplete:** Identify affected figures and which records or balances
  need attention. Do not present an unreliable value as complete.
- **Error:** Explain a failed calculation or data load and offer a retry or
  recovery path; do not silently substitute zeros.
- **Accessible charts:** Provide labels, legends, text summaries, keyboard
  operation, and non-color indicators for positive/negative status.
- **Responsive layout:** Preserve essential figures and actions across mobile
  and desktop layouts; explanations can collapse to reduce visual clutter.

## Number calculation checklist

Before a displayed figure is considered complete:

1. State its unit, currency, time period, and whether it is actual, planned,
   or projected.
2. State its formula and which transaction types are included or excluded.
3. Ensure transfers between the user's own accounts are not income or
   expenses.
4. Ensure savings and investments are not counted twice in net position or
   cash flow.
5. Reconcile the result to the underlying transactions, budget, schedule, or
   account balance.
6. Make corrections, refunds, reversals, uncategorized entries, and
   incomplete records visible in the calculation or explanation.
7. Provide a link to the supporting records.

## Acceptance checklist

- [ ] Users can view distribution for monthly, quarterly, half-yearly, and
  yearly periods.
- [ ] Circular charts and the monthly histogram use consistent figures
  derived from underlying records.
- [ ] Users can compare monthly income, expenses, savings, and investments.
- [ ] The monthly summary explains the position and significant changes.
- [ ] Each metric and chart can be traced to supporting transactions or
  financial records.
- [ ] Recurring payments show known amounts, frequencies, due dates, and
  available statuses without assuming a payment happened.
- [ ] The purchase projector explains estimated impact without creating a
  transaction.
- [ ] Transfers and overlapping categories do not inflate totals.
- [ ] Missing data, zero-value periods, refunds, and incomplete records have
  clear behavior.
- [ ] Calculations use consistent periods, currency formatting, and
  definitions.
- [ ] Actual results are distinguishable from estimates and projections.
- [ ] The dashboard works on mobile and is accessible.

## Open product decisions

1. Confirm the definition of the donut chart and how deficit is represented.
2. Confirm whether surplus is before or after savings and investment
   contributions; this document proposes net cash flow before those
   allocations.
3. Define overall and category budget utilization when both are configured.
4. Define how refunds and reversals change expense totals and historical
   comparisons.
5. Define forecast horizon, expected-income treatment, account selection, and
   data-quality thresholds for the purchase projector.
6. Confirm how changing a recurring amount affects the monthly equivalent and
   historical amount-change alert.

## Related specifications

- [Core Screen Design](./Core_Screen_Design.md)
- [Overview Summary Definitions](./Overview_Summaries.md)
- [Purchase-check Recommendations](./Purchase_Check_Recommendations.md)
