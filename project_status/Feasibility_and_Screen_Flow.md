# Logbook — Feasibility and Screen Flow

This document responds to [GitHub issue #9: Define the primary user journeys
and the screens needed to complete them](https://github.com/KomeOn/Python_CA/issues/9).
It uses the issue's full screen list, financial concepts, global experience
rules, navigation proposal, and definition of a complete journey. It is a
platform-independent product design, not a choice of programming language or
UI framework.

## Feasibility

**Feasible, with a staged implementation.** The issue describes a coherent
personal-finance application. Its core recordkeeping and manual purchase
planning can be built incrementally. However, the full screen list includes
substantial features—investment valuation, debt schedules, notifications,
reports, audit history, account recovery, and protected financial data—that
make this more than a small transaction logbook.

The current repository contains a prototype of purchase-affordability rules
and tests, plus early record and onboarding work. It does not yet implement
the complete issue's application screens, persistent end-to-end data flows,
authentication, or the whole financial model. Therefore, the requirements
are feasible, but the current code should not be treated as a finished or
production-ready financial application.

### Recommended delivery stages

1. **Design the user journeys:** Complete the screen and interaction
   specifications in this document, including empty states, validation,
   correction flows, and responsive navigation.
2. **Deliver a dependable manual-entry foundation:** Protect user accounts,
   manage accounts and balances, record core transactions, and provide
   consistent actual-versus-planned summaries.
3. **Add planning and the purchase checker:** Add recurring items, budgets,
   goals, and planned purchases; calculate and explain checks using shared
   records.
4. **Expand financial coverage:** Add investment and debt tracking, reports,
   reminders, and the richer transaction types when their accounting rules
   and data requirements have been specified.
5. **Harden and validate:** Test data integrity, privacy, recovery,
   accessibility, and responsive behavior before relying on the app for
   sensitive records.

This staging sequences delivery; it does not silently remove the screens or
requirements listed in issue #9. Any decision to omit or defer a requirement
from the milestone needs to be reflected in its scope and acceptance criteria.

### Feasibility conditions and product risks

- **One consistent financial model:** Define how actual transactions,
  planned commitments, allocations, and projections affect balances and
  summaries. Use one source of truth across screens.
- **No double counting:** Transfers between accounts are not income or
  expenses. A savings allocation does not create money. Paid bills must no
  longer be deducted as unpaid commitments. An investment sale is not wholly
  profit.
- **Recommendations need explicit policy:** The issue gives a broad
  purchase-capacity formula and multiple possible outcomes. Before
  implementation, define eligible funds, protected allocations, priorities,
  confidence, thresholds for caution, and whether expected income is included.
  Do not treat an estimate as a guarantee or financial advice.
- **Data provenance and freshness:** Show when balances and valuations were
  last confirmed, and whether a figure is recorded, estimated, or projected.
  Missing data should produce a guided setup or an explicit incomplete result,
  not a misleading zero.
- **Sensitive data:** Authentication, access controls, secure storage and
  transmission, backups, recovery, and account deletion need deliberate
  design. Do not connect to a financial institution without an appropriate
  authorization and data-access explanation.
- **Complexity belongs in later stages:** OCR, live market prices, institution
  integrations, tax calculations, automatic transfers, and purchases are not
  necessary for a useful manual-entry first release. Extracted or imported
  information must be reviewed before it changes a record.

## Screen map

The screen names below follow issue #9. Repeated forms and detail views are
shown as part of their parent section.

| Screen | Main purpose | Primary actions |
|---|---|---|
| Login / Sign Up | Protect personal financial data and provide account access | Sign up, log in, log out, recover access |
| Onboarding and Initial Financial Setup | Establish an honest starting financial picture | Set preferences, add accounts and balances, optionally add records and goals |
| Overview / Dashboard | Understand current and projected position | Review summaries, choose a period, drill into records |
| Transactions — List and Form | Record and correct financial activity | Add, edit, duplicate, delete, filter, link related records |
| Financial Accounts and Balances | Understand where money is held and reconcile it | Add account, set or adjust balance, review linked activity |
| Recurring Income and Bills | Compare expected income and bills with actual activity | Add schedule, mark or link payment, review upcoming and overdue items |
| Budgets and Spending Limits | Set and monitor period and category limits | Create budget, adjust limits, review spent and projected amounts |
| Savings Goals | Track targets, allocations, and contributions | Create goal, record contribution or withdrawal, review progress |
| Investments | Track holdings and investment activity | Add holding or transaction, update valuation, review performance |
| Planned Purchases | Prepare and track intended purchases | Add purchase plan, set target and priority, update status |
| Purchase Checker | Evaluate a proposed purchase using shared financial data | Enter cost and date, review result and assumptions |
| Debts and Liabilities | Track amounts owed and repayment obligations | Add liability, record repayment, review balance and schedule |
| Reports and Financial Analytics | Explore historical and comparative information | Select report, period, and filters; export |
| Notifications and Reminders | Review important upcoming or changed items | Open alert, follow its action link, update preferences |
| Settings and Data Management | Manage preferences, privacy, and user records | Update preferences, export, back up, restore, or delete data |

## Navigation

Use the issue's primary sections, grouping planning features together. On
small screens, show the primary sections in bottom navigation and place
secondary sections in a menu. On larger screens, use a sidebar or equivalent.

```text
Primary navigation
├── Overview
├── Transactions
├── Planning
│   ├── Recurring Income and Bills
│   ├── Budgets
│   ├── Savings Goals
│   ├── Planned Purchases
│   └── Debts and Liabilities
├── Investments
└── Purchase Checker

Secondary navigation
├── Financial Accounts and Balances
├── Reports and Financial Analytics
├── Notifications and Reminders
└── Settings and Data Management
```

Related detail and edit forms open from the screen that owns the record and
return there when finished. A user should be able to open a summary, see the
records behind it, correct a record, and understand how that change affects
the displayed position.

## Core end-to-end flow

```text
Sign up / Log in
  -> Onboarding and initial financial setup
  -> Overview
  -> Record or correct financial activity
  -> Review accounts, budgets, goals, investments, and debts
  -> Plan commitments or a future purchase
  -> Check the purchase against current/projected finances
  -> Review assumptions and any suggested action
  -> Confirm and record any action taken; otherwise leave the plan unchanged
  -> Review updated position, reports, and reminders
```

The checker only evaluates and explains. It must not execute a transfer,
withdrawal, sale, or purchase. An action is not complete until the user
confirms it and the resulting financial activity is recorded.

## Detailed user journeys

### 1. Sign up and establish the initial financial baseline

```text
Login / Sign Up
  -> Create account or log in
  -> Onboarding
  -> Choose currency, locale, date format, and financial preferences
  -> Add financial accounts and opening balances
  -> Optionally record recurring income, bills, savings, investments, debts,
     budgets, and planned purchases
  -> Set minimum cash / emergency reserve
  -> Review setup checklist
  -> Overview
```

- Returning users go to Overview. New users go to onboarding.
- A user with incomplete setup can resume without losing completed steps.
- Optional setup steps can be skipped and completed later.
- Explain which summaries and checks are unavailable or less reliable when
  information is missing. Do not invent a balance or show missing information
  as a real zero.
- Record the currency and date context so amounts and dates are interpreted
  consistently.

### 2. Review the Overview

```text
Overview
  -> Select a reporting period
  -> Choose whether to inspect actual, planned, or projected information
  -> Review cash, savings, investments, debts, income, expenses, and surplus
  -> Review upcoming obligations, goals, purchases, and alerts
  -> Select a figure or chart
  -> Open its supporting records or relevant planning screen
```

- Summaries use the same definitions and underlying records as the related
  screens.
- Distinguish current recorded values from estimates and future projections.
- Provide charts only where there is enough data to make them meaningful.
- New users see a setup checklist instead of empty charts or misleading totals.
- Do not count internal transfers as income or expenses, or add a goal
  allocation on top of the account balance holding that money.

### 3. Record, find, and correct financial activity

```text
Transactions
  -> Search or filter by date, category, account, amount, type, tag, or status
  -> Add, duplicate, or open a transaction
  -> Enter type, date, amount, category, account, and relevant details
  -> Optionally add notes, tags, payment method, receipt, or linked record
  -> Validate and review
  -> Save
  -> Updated transaction list and dependent summaries
```

- The list is reverse chronological by default. Each entry shows date and
  time, description, amount and currency, type, category, account, and status
  where relevant.
- Supported transaction types in the issue include income, expense,
  account-to-account transfer, refund or reversal, debt received or repaid,
  investment purchase or sale, investment income, and savings movement.
- A transfer changes account balances but is not income or expense. Record
  refunds, split transactions, partial payments, and links to bills, goals,
  investments, debts, or planned purchases without counting the same event
  twice.
- When a receipt is captured or OCR is used, show extracted details for user
  review and confirmation before saving.
- Distinguish pending from settled activity, flag possible duplicates, and
  preserve enough edit history to explain changes.
- Before deletion or reversal, explain the effect and ask for confirmation.
  Historical summaries must not silently become incorrect.

### 4. Review and reconcile financial accounts

```text
Financial Accounts and Balances
  -> Open an account or add one
  -> Review account type, currency, recorded balance, last confirmed date,
     linked transactions, and reconciliation status
  -> Enter opening balance, adjustment, or manually confirmed balance
  -> Compare with recorded activity
  -> Review and confirm any adjustment
  -> Updated account and dependent summaries
```

- Support the account types listed in the issue: cash/wallet, bank, savings
  or deposit, credit card, brokerage/investment, loan, and other relevant
  accounts.
- Clearly label recorded and available balances where both are meaningful.
- Keep account balances separate from goal allocations. Money assigned to a
  goal remains in its financial account and is not another asset.
- Show the balance confirmation date so stale data is visible to summaries
  and the purchase checker.

### 5. Manage recurring income and bills

```text
Planning -> Recurring Income and Bills
  -> Review expected amount, schedule, next date, and status
  -> Add or edit an income or bill schedule
  -> Review next occurrence and save
  -> As the date arrives, mark received/paid or link an actual transaction
  -> Compare expected and actual amount
  -> Update forecasts and outstanding commitments
```

- Show expected versus actual amounts and statuses such as received,
  pending, delayed, missed, paid, upcoming, overdue, or partially paid.
- Support variable bills and irregular income; do not imply an estimate is a
  confirmed amount or event.
- When a bill is paid, link or create its actual transaction and remove that
  occurrence from outstanding commitments. Do not deduct it twice.
- Distinguish expected income from received income. Categorize refunds, loan
  disbursements, investment proceeds, and repayments according to their
  nature, not automatically as recurring income.
- Before changing a recurring schedule, explain which future occurrences
  will change. Do not silently edit historical transactions.

### 6. Create and monitor budgets

```text
Planning -> Budgets
  -> Create a monthly or custom-period budget
  -> Set overall and/or category spending limits
  -> Review budgeted, spent, remaining, and projected end-of-period amounts
  -> Receive or review threshold warning
  -> Open the transactions contributing to a figure
  -> Adjust a limit and retain its change history
```

- State which transaction types count as spending in each budget.
- Transfers, savings allocations, and investment purchases are not ordinary
  consumption expenses by default.
- Show comparisons with prior periods only when comparable records exist.
- Distinguish actual spending from a projected end-of-period amount.

### 7. Track savings goals and allocations

```text
Planning -> Savings Goals
  -> Add a goal with target, target date, and optional priority
  -> Record planned contribution schedule
  -> View allocated/saved, remaining, progress, and time remaining
  -> Link actual contributions or withdrawals
  -> Review planned versus actual progress and estimated completion
```

- Creating a goal does not create money.
- Keep planned contributions separate from actual contributions.
- Allocating cash to a goal is not new income, and must not add another asset
  on top of the account balance.
- A withdrawal reduces the associated allocation.
- Do not let multiple goals reserve the same funds unless shared allocation
  is deliberately supported and explained.

### 8. Track investments

```text
Investments
  -> Add a holding or investment transaction
  -> Enter supported asset details, quantity, cost basis, fees, and valuation
  -> Record the valuation date and source
  -> Review portfolio value, allocation, history, and realized/unrealized
     results
  -> Update a manual valuation or linked transaction
```

- The issue calls for holdings, transactions, performance, and valuation
  details. These require a defined investment data model and must not be
  mistaken for immediately spendable cash.
- Distinguish realized from unrealized gain/loss; treat purchases and sales as
  investment transactions, not ordinary consumption.
- Treat dividends and interest as investment income; do not treat sale
  proceeds as all profit.
- Show data source and valuation date; account for fees, taxes, settlement,
  and exit restrictions where known. Do not imply returns or liquidity are
  guaranteed.
- Allow manual entry if live market data is not available.

### 9. Plan a future purchase and check affordability

```text
Planning -> Planned Purchases
  -> Add purchase name, estimated total cost, target date, and priority
  -> Optionally link a savings goal and planned contributions
  -> Review funding progress and update the plan
  -> Open Purchase Checker
  -> Enter or confirm total cost, additional costs, and proposed date
  -> Review result, projected position, assumptions, and alternatives
  -> Choose whether to change the plan
  -> If purchased, link the actual transaction and update status
```

- A purchase plan is not an actual expense. A saved amount is not extra cash
  if it is already in an account balance.
- Track plan status as planned, saving, ready, postponed, purchased, or
  cancelled; update remaining cost when price or contributions change.
- The checker uses shared financial records, not a separate set of balances.
- The issue's proposed capacity formula is:

  ```text
  Purchase Capacity =
      Eligible Funds
      - Outstanding Commitments
      - Protected Allocations
      - Planned Contributions
      - Minimum Reserve

  Purchase Affordability = Purchase Capacity - Total Purchase Cost
  ```

- For a future date, project the relevant values to that date and deduct only
  commitments that have not already been reflected in recorded balances.
- Display purchase capacity, shortfall or remaining margin, current and
  projected surplus, post-purchase funds, effects on reserves/goals/
  investments/bills, data freshness, assumptions, and confidence.
- Possible outcomes in the issue include Affordable — Proceed, Affordable
  with Caution, Save First, Postpone Purchase, Reduce Purchase Cost,
  Reallocate Eligible Funds, Not Recommended, and Insufficient Data. Before
  implementation, define rules that make these outcomes mutually
  understandable, especially how limited buffer, priorities, and data
  confidence affect the recommendation.
- A positive result is not automatically advice to buy. Never execute a
  transfer, withdrawal, sale, or purchase automatically. Require user
  confirmation and record an action if it is taken.

### 10. Track debts and liabilities

```text
Planning -> Debts and Liabilities
  -> Add liability or money owed to the user
  -> Enter original and outstanding amount, rate/fees, due dates, and schedule
  -> Record repayments, interest, fees, or money lent/returned
  -> Review balance, history, and upcoming payments
  -> Update forecast and purchase-check commitments
```

- Support personal loans, home/vehicle/education loans, credit-card
  liabilities, and money borrowed from or lent to others.
- Where known, distinguish principal repayment from interest and fees.
- Borrowing is not income; lending is not an ordinary expense.
- Include repayment obligations in cash-flow forecasts without duplicating a
  bill or transaction already represented elsewhere.

### 11. Review reports and financial analytics

```text
Reports
  -> Select a report and period
  -> Filter by date, category, account, or other supported dimension
  -> Review historical or comparative results
  -> Drill into supporting records
  -> Export a report if needed
```

- The issue lists income/expense statements, cash flow, category/merchant
  spending, budget performance, savings rate and goals, investment
  performance, debt summaries, net-worth history, and actual-versus-projected
  position.
- Historical reports use records and valuations applicable to the selected
  period. Never present an estimated historical balance as an actual
  confirmed balance.
- Exports need privacy safeguards and clear labels for actual and estimated
  values.

### 12. Review notifications and reminders

```text
Notification Center
  -> Review unread and read items
  -> Identify informational messages versus urgent warnings
  -> Open the related account, bill, budget, goal, purchase, or transaction
  -> Resolve or dismiss the item
  -> Configure thresholds, frequency, and notification preferences
```

Potential reminders include upcoming/overdue bills, expected income not
received, low cash or reserve, budget thresholds, goal progress, purchase
milestones, investment valuation changes where supported, debt repayments,
stale data, and possible duplicates. Label uncertain forecasts as estimates,
not confirmed events. Status must not depend on color alone.

### 13. Manage settings and personal data

```text
Settings and Data Management
  -> Update profile, currency/locale/date, categories/tags, periods,
     reserve preferences, accessibility, and notifications
  -> Export records or reports
  -> Back up or restore where supported
  -> Review synchronization or account connections, if present
  -> Correct or delete records
  -> Request account and associated data deletion
```

- Explain what data is stored and what any connection can access.
- Protect sensitive financial information in storage and transit.
- Confirm consequential actions and account deletion; explain their effects.
- Report save, import, export, restore, and synchronization errors explicitly.

## Shared screen and interaction rules

Apply these issue #9 requirements throughout the application:

1. **Single source of truth:** A financial event and its calculations agree
   on every screen.
2. **Actual versus planned:** Label recorded transactions, scheduled events,
   forecasts, and proposed actions distinctly.
3. **No double counting:** Apply consistent treatment to transfers, savings,
   investments, debts, and commitments.
4. **Data freshness:** Show when balances and valuations were last confirmed.
5. **Transparent recommendations:** Explain the figures, assumptions, and
   trade-offs behind significant recommendations.
6. **Useful empty states:** Guide setup rather than show misleading zeros.
7. **Clear errors:** Explain validation and failed save/import/calculation/
   synchronization, and provide a next step where possible.
8. **Auditability:** Preserve enough transaction and change history to
   explain balances and recommendations.
9. **Responsive layout:** Support desktop, tablet, and mobile without hiding
   essential information or actions.
10. **Accessibility:** Use readable typography, clear labels, keyboard
    navigation, and indicators that do not rely on color alone.
11. **Privacy and confirmation:** Protect personal data and confirm destructive
    or consequential actions.
12. **No implied execution:** A plan or recommendation is not a completed
    financial action until the user confirms and records it.

## Definition of a complete journey

The design is complete when a user can create an account and initial
financial baseline; record and reconcile financial activity; track recurring
items, budgets, goals, investments, and debts; plan purchases; check a
purchase against current or projected finances; understand the assumptions
and trade-offs; review reports and reminders; and correct records without
inconsistent balances, duplicate commitments, or unexplained
recommendations.

## Decisions to settle before implementation

- Which features are required in the first usable release, and which are
  explicitly deferred?
- Which account and transaction types are included initially?
- How are transfers, refunds, debt activity, savings allocations, and
  investment transactions represented in the shared ledger?
- Which funds are eligible for a purchase check, and how are protected
  allocations and competing goals handled?
- Which future income and variable expenses are forecast, and how is their
  confidence presented?
- What thresholds distinguish Affordable with Caution, Save First, Postpone,
  Reduce Cost, Reallocate Funds, and Not Recommended?
- How are currencies, dates, time zones, valuations, and stale data handled?
- What authentication, storage, backup, recovery, and deletion guarantees
  can the chosen deployment actually provide?
- Which reminders are in-app only, and which require delivery outside the
  application?

For the current MVP affordability behavior and scenarios, see
[Budget Rules](./Budget_Rules.md) and
[Purchase-check Recommendations](./Purchase_Check_Recommendations.md).
Those existing documents define a narrower rule set; reconcile them with the
broader issue #9 purchase-capacity policy before implementation.
