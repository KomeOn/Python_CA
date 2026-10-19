# Logbook — Transaction List and Transaction Form

This specification is based on [GitHub issue #11: Specify the transaction
list and the create/edit transaction form](https://github.com/KomeOn/Python_CA/issues/11).
It describes the transaction list, manual entry, optional image/PDF entry,
and the interactions needed to keep saved records reliable. It does not
implement these screens.

## Objective

Let users find, view, create, edit, and delete financial transactions.
Transactions can be entered manually or prepared from an image/PDF and then
reviewed and confirmed by the user before saving.

Income, expenses, and transfers must remain distinguishable in both the
interface and financial calculations. A transfer between the user's own
accounts is not income or an expense.

## Transaction list

### Default list

Show the 10 most recent transactions, ordered newest first by transaction
date and time. Include an action to add a transaction and a way to reach the
full history.

```text
Transactions                              [Add transaction]
[Search description, merchant, sender...]
[Type v] [Category v] [Account v] [Date range] [Amount range]

Date / time       Type       Description       Category      Account     Amount
...               Expense    ...               ...           ...         ...
...               Income     ...               ...           ...         ...
...               Transfer   ...               ...           ...         ...

[Load more / Full history]
```

Each row shows the available:

- Transaction date and time.
- Type: income, expense, or transfer.
- Amount and currency.
- Sender, receiver, merchant, or description, as applicable.
- Category, if assigned.
- Account or bank name, optionally hidden or shown according to the user's
  display preference.

Use labels, signs, or accessible icons to distinguish transaction type and
financial impact. Do not rely on color alone. If counterparty or description
is unavailable, display a clear fallback such as “No description.”

Selecting a row opens complete transaction details. Provide a convenient
edit action on each transaction. Older records are reachable through
pagination, “Load more,” or a full transaction history view.

### Search and filters

Support:

- Text search across sender, receiver, merchant, and description.
- Transaction type.
- Category.
- Account or bank.
- Date range.
- Optional amount range.

The default remains the latest 10 transactions. Searching and filtering
must include older matching records, not just the currently loaded rows.
Show active filters and provide a clear way to remove them. If there are no
matches, explain that and offer to clear filters or add a transaction.

### Transaction actions

- **Add transaction:** Opens a blank form.
- **Open details:** Shows all saved fields and available actions.
- **Edit:** Opens the selected transaction with its existing values.
- **Delete:** Requires confirmation and explains whether deletion is
  permanent or recoverable.

If there is a list-level edit action, it must first let the user select which
transaction to edit. A successful change updates the list and affected
financial summaries. A failed change leaves the saved transaction intact,
explains the error, and allows retry.

## Manual transaction form

Clearly identify whether the user is creating a new transaction or editing
an existing one.

### Fields

| Field | Requirement | Behavior |
|---|---|---|
| Transaction type | Required | Income, expense, or transfer; choosing a type changes the relevant account and counterparty fields |
| Amount | Required | Accept a valid decimal amount using the configured currency precision |
| Currency | Required | Default to the user's configured currency where appropriate; allow changes if supported |
| Transaction date | Required | Default sensibly, but allow correction |
| Transaction time | Optional | Show when supported |
| Sender, receiver, merchant, or description | Optional | Label according to type; show an understandable fallback in list/details if absent |
| Source account | Optional, depending on type and whether accounts are tracked | Required for a transfer |
| Destination account | Optional, depending on type and whether accounts are tracked | Required for a transfer |
| Category and subcategory | Optional | Allow uncategorized records |
| Notes | Optional | Preserve when editing unrelated fields |
| Reference/external transaction ID | Optional | Retain if provided |
| Attachments | Optional, where supported | Keep associated with the transaction according to the product's data policy |

### Type-specific fields

**Income**

- Amount received.
- Optional payer/source.
- Optional destination account.
- Optional category and notes.

**Expense**

- Amount spent.
- Optional merchant/receiver.
- Optional source account.
- Optional category and notes.

**Transfer**

- Amount transferred.
- Source account.
- Destination account.
- Optional date/time and notes.
- Source and destination must be distinct accounts.
- The transaction must be represented as a transfer, not as income plus an
  expense.

### Form controls and validation

```text
Add transaction
Type: [Income v]
Amount: [              ] Currency: [Configured currency v]
Date: [                ] Time (optional): [       ]
Payer / merchant / description: [                 ]
Account: [                                          ]
Category: [Uncategorized v]
Notes (optional): [                                 ]
Attachment (optional): [Choose file]

[Cancel]                                  [Save transaction]
```

The exact fields adapt to the selected type. In edit mode, populate the
existing values and preserve fields the user does not change.

- Validate required fields, date, amount, currency precision, and transfer
  accounts.
- Reject invalid or non-finite amounts and invalid dates.
- Place actionable validation messages beside the relevant fields.
- Preserve all entered values when validation fails.
- Allow the user to save or cancel.
- Warn before discarding unsaved changes.
- Prevent repeated save actions from creating duplicate submissions.
- Show a success confirmation and return to the transaction list or saved
  transaction details.

Categories may be selected or left uncategorized. If category management
allows a category to be renamed or changed, existing transactions must remain
associated with the correct category. Editing another field must not clear
the category or notes.

## Image or PDF entry

The user can optionally start transaction entry by selecting an image or PDF
of a receipt, invoice, statement entry, or payment confirmation.

```text
Transactions
  -> Add transaction
  -> Choose manual entry or upload image/PDF
  -> Select document
  -> Validate file and show processing state
  -> Extract possible transaction details
  -> Show editable form with extracted values and uncertainty
  -> User corrects and reviews all fields
  -> User explicitly confirms Save
  -> Duplicate warning, if a possible match exists
  -> Save transaction or return to edit
```

### Upload and processing

- Define supported image/PDF formats and configured limits for file size and
  PDF page count.
- Show the selected file and allow removal or replacement before saving.
- Show a processing/loading state while information is extracted.
- Explain unsupported, corrupted, unreadable, and password-protected file
  errors, with a recovery action.
- If a file contains multiple transactions, present them as separate
  candidate records for review. Never silently combine them into one.

### Extracted values and confirmation

Extraction may find amount, currency, transaction date/time, counterparty or
description, reference, account/bank details, and transaction type when that
type can be inferred reliably. Category, subcategory, and notes may be
suggested, but must not be presented as facts explicitly found in the
document unless they were.

Before saving:

- Show extracted values in the regular editable transaction form.
- Let the user correct every field.
- Indicate missing, ambiguous, or low-confidence values where supported.
- Do not silently substitute an uncertain amount, date, currency, or type.
- Require an explicit user action to confirm saving.
- Allow cancellation without creating a transaction.
- Keep the original document available for review during confirmation where
  supported.
- Processing completion alone must never create a transaction.

### Possible duplicate warning

When a document may describe an existing transaction, show a warning and
allow the user to compare the candidate with possible matches. Use available
reference, amount, currency, date, account, and merchant information.

The user can continue saving if the transactions are legitimately distinct.
Do not automatically discard or suppress a candidate solely because it
resembles another record.

## Data integrity and cross-screen behavior

- Save edits only after validation succeeds; preserve unchanged fields.
- Clearly report save failures and allow retry without losing entered data.
- Refresh all affected balances, summaries, charts, and category budgets after
  an edit or deletion.
- Preserve enough change history and timestamps to explain edits, where audit
  history is supported.
- Prevent a user from accessing or editing another user's transaction.
- Handle simultaneous edits safely; do not silently overwrite another
  update.
- Confirm deletion and make clear whether the transaction is permanently
  deleted or recoverable.
- Ensure transfers, refunds, reversals, and other types are treated
  consistently with the shared financial definitions.

## Screen states and accessibility

- **Loading:** Indicate when history or a document is being processed.
- **Empty list:** Explain that no transactions exist and offer Add transaction.
- **No search results:** Explain that filters found no matches and offer to
  clear them.
- **Validation error:** Identify the field and how to correct it without
  clearing other values.
- **Upload/processing error:** Identify the problem and offer to replace or
  retry the file.
- **Save error:** Preserve form values and explain how to retry.
- **Success:** Confirm the transaction was saved and return to the list or
  details.
- Use readable labels, accessible field descriptions, keyboard navigation,
  and non-color indicators for type and financial impact.
- Keep the list and form usable on desktop and mobile layouts.

## Acceptance checklist

- [ ] The default list shows 10 most recent transactions in descending date
  and time order.
- [ ] Older transactions can be reached, searched, and filtered by the
  supported criteria.
- [ ] Rows show the available date, amount/currency, type, and
  counterparty/description.
- [ ] Users can create, view, edit, and delete records with clear deletion
  behavior.
- [ ] Required fields are validated, errors are actionable, and form input is
  preserved after validation failure.
- [ ] Income, expenses, and transfers are represented consistently; internal
  transfers are not counted as income or expenses.
- [ ] Supported images/PDFs can be uploaded and extracted details reviewed
  before saving.
- [ ] Uncertain or missing extracted values are not silently treated as
  verified.
- [ ] Possible duplicates are reviewed with the user, not automatically
  suppressed.
- [ ] Edits and deletions update affected summaries and budgets.
- [ ] Upload, processing, validation, and save failures are clear and
  recoverable.
- [ ] The list and form are accessible and usable on supported screen sizes.

## Out of scope unless separately approved

- Direct bank synchronization or importing transactions from financial
  institutions.
- Executing payments automatically.
- Saving extracted transactions without user confirmation.
- Advanced receipt accounting, tax calculations, or invoice management.
- Automated categorization that the user cannot review.

## Related specifications

- [Core Screen Design](./Core_Screen_Design.md)
- [Core User Journeys](./User_Journeys.md)
- [Budget Overview Dashboard](./Budget_Overview_Dashboard.md)
