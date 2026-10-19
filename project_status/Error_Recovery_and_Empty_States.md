# Logbook — Error Recovery, Recurring Items, and Empty States

This design review is based on [GitHub issue #13: Review the designs for
correcting mistakes, changing recurring items, and handling empty or
incomplete data](https://github.com/KomeOn/Python_CA/issues/13). It defines
screen flows and behavior for corrections, recurring schedule changes, and
data states. It does not implement those behaviors.

## Goals

- Help users safely correct or remove mistaken information.
- Explain what an edit to a recurring item will change before it is saved.
- Distinguish scheduled occurrences from completed transactions.
- Make empty, filtered, incomplete, loading, and error states understandable
  and actionable.
- Keep status and interaction patterns consistent, accessible, and
  responsive.

## 1. Correcting a transaction or other record

### Edit flow

```text
List, chart, or summary
  -> Open the record
  -> Choose Edit
  -> Edit form opens with saved values
  -> Change fields
  -> Save
       | Valid                       | Invalid / save fails
       v                             v
  Save changes                   Keep entered values
  Recalculate affected           Explain problem and
  summaries/lists                 provide correction or retry
       |
       v
  Show success and updated record
```

- Forms clearly identify the selected record and prefill its existing values.
- Validation errors appear beside the relevant field, explain how to correct
  the value, and do not clear other entered changes.
- A save failure is distinct from validation failure. Preserve the user's
  edits and offer retry; do not display success or silently discard changes.
- On success, confirm the save and return to the record details or list.
  Refresh affected balances, summaries, charts, budgets, and purchase
  calculations.
- Editing a transaction corrects that transaction. It must not silently
  change the recurring template that may have produced it.

### Unsaved changes flow

```text
Modified form
  -> User chooses Cancel, Back, or another destination
  -> Prompt: "Discard unsaved changes?"
       | Keep editing                | Discard changes
       v                             v
  Return to form                  Leave form
                                  Do not save changes
```

Show this prompt only when there are unsaved modifications. The user must be
able to remain on the form or explicitly discard; navigating away must not
silently lose entered information.

### Delete and recovery flow

```text
Record details
  -> Choose Delete
  -> Confirmation identifies record and deletion behavior
       | Cancel                     | Confirm
       v                             v
  Keep record                     Delete or move to recovery
                                  Show success and update summaries
```

- The confirmation states whether deletion is permanent or recoverable and
  identifies the record being removed.
- If the product supports recovery, provide a clear route and explain the
  recovery window or relevant limits.
- If deletion is permanent, say so before confirmation. Do not claim a record
  can be recovered if it cannot.
- Removing or restoring a record recalculates every affected list and
  financial summary.
- Use confirmation for consequential changes. Avoid extra confirmations for
  harmless navigation.

## 2. Recurring item changes

Recurring items include subscriptions, bills, recurring income, and other
scheduled entries. The detail view should show the name, amount, frequency,
next scheduled date, current status, and whether an occurrence is scheduled
or already represented by an actual transaction.

### Create or edit flow

```text
Recurring items
  -> Add item or open an existing item
  -> Enter/review name, amount, frequency, next date, account, and category
  -> Save
  -> Review next scheduled occurrence and updated list
```

When editing a series, ask which occurrences the change should affect:

```text
Apply this change to:
( ) Current occurrence only
( ) Current and future occurrences
( ) Entire series (where supported)

[Cancel] [Review changes]
```

Before applying, summarize the effect in plain language. For example:
“Change the amount from 25 to 30 for this and future unpaid occurrences
starting 1 November.” The user confirms the scope and saves, or returns to
edit.

- **Current occurrence only:** Change just the selected scheduled occurrence;
  leave the recurring template and other dates unchanged.
- **Current and future occurrences:** Change the selected occurrence and
  later occurrences; preserve completed historical transactions.
- **Entire series:** Use only when supported and explicitly explain which
  historical and future records change. Do not silently rewrite posted
  transactions; use correction records or an audit trail if historical
  changes are permitted.

The issue requires these choices but does not define whether the application
will support historical series edits. Confirm that policy before
implementation. If an option is unsupported, do not display it.

### Pause, resume, and cancel

```text
Recurring item details
  -> Pause / Resume / Cancel series
  -> Explain status change and effect on future occurrences
  -> Confirm when the action stops or removes future occurrences
  -> Update item status, upcoming lists, reminders, and projections
```

- **Paused:** Temporarily stops generating or presenting future occurrences
  as active obligations; preserves history and the ability to resume.
- **Resumed:** Continues the schedule from a clearly shown next date. Ask for
  a new next date if it cannot be derived unambiguously.
- **Cancelled:** Ends future occurrences from the effective date. Confirm
  cancellation and explain that past completed transactions remain in
  history.
- Cancellation must not delete completed transactions.
- Never count both a completed transaction and its scheduled occurrence.
- Display statuses with text/icons as well as color: completed, upcoming,
  paused, cancelled, and overdue, where applicable.
- A scheduled date passing does not prove that a payment was made. Only mark
  completed when the user or a trusted source confirms it.
- Changes update the recurring list, reminders, summaries, and projections
  consistently.

## 3. Empty, incomplete, loading, and error states

Use distinct messages and actions for the following states; they are not
interchangeable.

| State | Meaning | User-facing response |
|---|---|---|
| **Genuinely empty** | No records exist for this feature or dataset | Explain what the feature is for and offer the relevant first action, such as Add transaction, Create budget, or Add recurring item |
| **No search results** | Records exist, but current search or filters match none | Say no records match the current criteria; show active filters and offer Clear filters |
| **Incomplete data** | Some records exist, but required fields or balances are missing, unverified, or insufficient | Identify the missing information, which figures are affected, and a direct action to review or complete it; label any usable partial result |
| **Loading** | Data is being retrieved, calculated, or processed | Show a non-misleading progress state; do not briefly present zero as the result |
| **Load or calculation error** | The application could not retrieve or calculate the result | Explain what failed, preserve the selected context, and provide Retry; do not show a success-shaped fallback |
| **Zero-value result** | Data is present and the valid calculation equals zero | Show zero as a calculated result and explain its basis; do not treat it as missing data |

### Screen-specific state guidance

| Screen/component | Empty state | Incomplete or error behavior |
|---|---|---|
| Transaction list | “No transactions yet” with Add transaction | Preserve search/filters on load error; distinguish no matches from no saved transactions |
| Budgets | “No budgets configured” with Create budget | Do not treat unbudgeted spending as zero spending; label budget utilization unavailable |
| Graphs and dashboard summaries | Explain which records are needed before showing a chart | Identify missing categories, history, or balances; avoid fabricated zero series or confident trend statements |
| Recurring items | “No recurring items yet” with Add recurring item | Show unknown payment status as unknown; distinguish unavailable schedule data from no schedule |
| Search results | “No matching records” and Clear filters | If the search failed, show an error and Retry instead of an empty result |
| Purchase checker | Guide the user to enter required purchase details | Identify missing, stale, or unverified data and suppress unreliable conclusions |

For all screens, keep the user's selected period, search, and navigation
context when retrying or resolving an issue where practical.

## 4. Data certainty and financial summaries

- Label actual transactions separately from scheduled items, estimates, and
  projections.
- Identify missing, unavailable, unverified, or stale values where they affect
  a result.
- Do not present an incomplete amount as a confirmed financial result.
- If a partial calculation is useful, label it as partial and state what is
  excluded; otherwise withhold the result and explain what needs attention.
- A calculation error must never be replaced silently by zero, a previous
  result, or a positive recommendation.
- Graphs must handle zero values, missing categories, and insufficient history
  without implying data that was not recorded.
- Changes to a record update dependent summaries consistently and remain
  traceable to their supporting records.

## 5. Consistency and accessibility

- Reuse consistent labels, field placement, icons, buttons, and status
  patterns across record types.
- Make errors and statuses understandable without color alone; pair color
  with text, symbols, or accessible labels.
- Provide keyboard-accessible forms, controls, menus, and confirmation
  dialogs, with screen-reader labels and descriptions.
- Put focus in a confirmation dialog when it opens and return focus to the
  initiating control when it closes.
- Explain the consequences of destructive and recurring-series actions
  before confirmation.
- Keep forms, state messages, and recovery actions usable on mobile and
  desktop.
- Ensure each exceptional state includes an understandable next step.

## 6. Usability gaps and decisions to resolve

1. **Deletion policy:** Decide which record types are recoverable, how
   recovery works, and whether a recovery period applies.
2. **Recurring series history:** Decide whether “entire series” can affect
   historical occurrences. If so, define auditability and how completed
   transactions are corrected without silently rewriting financial history.
3. **Current occurrence boundary:** Define what “current occurrence” means
   when a date has passed but payment status is unknown or the item is overdue.
4. **Partial data policy:** Decide which dashboard values may be shown as
   partial estimates and which must be withheld.
5. **Status sources:** Define how completed/overdue payment status is
   confirmed, especially when no bank integration is in scope.
6. **Error recovery:** Define whether a failed save or calculation can be
   retried safely without duplicating records or losing form state.

These are product decisions, not implementation assumptions. Track unresolved
decisions as follow-up work before building affected flows.

## Acceptance checklist

- [ ] Edit forms are prefilled and support correcting all relevant fields.
- [ ] Validation and save errors are field-specific or actionable and preserve
  entered values.
- [ ] Unsaved changes prompt before leaving a modified form.
- [ ] Destructive actions explain consequences and require confirmation.
- [ ] Deletion recovery is accurately described where supported.
- [ ] Recurring edits let users choose a supported scope and review its effect.
- [ ] Pause, resume, and cancellation are clear and update future occurrences
  without deleting completed transactions.
- [ ] Scheduled occurrences are distinguishable from completed transactions
  and cannot be double-counted.
- [ ] Relevant screens define empty, no-results, incomplete, loading, error,
  and zero-value behavior.
- [ ] Missing or unverified data does not become a misleading financial
  summary.
- [ ] Patterns support keyboard and screen-reader use, mobile and desktop,
  and non-color status communication.
- [ ] Design gaps and unresolved policies are documented or tracked.

## Related specifications

- [Core Screen Design](./Core_Screen_Design.md)
- [Core User Journeys](./User_Journeys.md)
- [Transaction List and Form](./Transaction_List_and_Form.md)
- [Budget Overview Dashboard](./Budget_Overview_Dashboard.md)
- [Overview Summary Definitions](./Overview_Summaries.md)
