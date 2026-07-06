# Invoice & Payment Management Tool — PRD
**Apple Music Publishing Operations | Take-Home Prototype**

## 1. Overview

An internal accounts payable tool for the Apple Music Publishing Operations team. Licensors (music rights holders) send Apple invoices for licensing fees owed across one or more territories and Apple Music service plans. This tool lets ops team members upload/create invoices, track what's owed, and process payments — fully or partially by territory/plan — with a full audit trail per invoice.

**This is accounts payable**: Apple is the payer, licensors are the payees.

## 2. Goals

- Give ops team members a single system of record for licensor invoices and payment status.
- Support partial payment at the territory/plan line-item level (e.g., pay the US/Individual line now, hold France/Family for later).
- Make payment history and invoice status transparent and auditable at a glance.
- Support paying multiple invoices/line items in a single batch action.

## 3. Data Model

### Licensor
| Field | Notes |
|---|---|
| id | |
| name | |
| point_of_contact | name + email |
| default_payment_terms | e.g., Net 30 — can be overridden per invoice |

### Invoice
| Field | Notes |
|---|---|
| id | |
| licensor_id | FK → Licensor |
| invoice_number | licensor-assigned, entered at upload |
| invoice_date | |
| payment_terms | e.g., Net 30; defaults from licensor, editable |
| currency | |
| total_amount | sum of line items |
| status | **Unpaid / Partially Paid / Paid / Blocked** — computed (see §5), except Blocked which is a manual override |
| source | CSV upload or manual entry |
| created_by / created_at | |

### Invoice Line Item
| Field | Notes |
|---|---|
| id | |
| invoice_id | FK → Invoice |
| territory | e.g., US, UK, DE |
| plan_type | Individual / Family / Student / etc. |
| amount | |
| line_status | Unpaid / Paid — full-or-nothing, no partial payment within a line |

### Payment
| Field | Notes |
|---|---|
| id | |
| line_item_id | FK → Invoice Line Item (payment always pays a line in full) |
| amount | = line item amount (denormalized for audit clarity) |
| payment_date | |
| processor | Apple Music Pub Ops team member who executed the payment |
| batch_id | groups payments executed together in one payment run (nullable) |
| notes | optional free text |

## 4. Screens

### 4.1 Invoice Upload & Creation
- Two paths: **CSV upload** (bulk) or **web form** (single invoice, manual entry).
- Manual form: licensor (searchable dropdown, or "add new licensor"), invoice number, invoice date, payment terms, currency, then a repeatable line-item block (territory, plan type, amount). Total auto-calculates from line items.
- CSV upload: template with columns for licensor, invoice number, date, terms, currency, territory, plan, amount (one row per line item, invoice-level fields repeated). Show a preview/validation step before committing — flag missing licensors, malformed rows, currency mismatches.
- On save: status initializes to Unpaid.

### 4.2 Invoice Browser
- Scrollable table: Invoice #, Licensor, Invoice Date, Due Date (derived from terms), Total Amount, Status, Outstanding Balance.
- Filters: status, licensor, territory, plan type, date range.
- Search: invoice number, licensor name.
- Status shown as a colored badge (Unpaid / Partially Paid / Paid / Blocked).
- Row click → Invoice Detail View.

### 4.3 Invoice Detail View
- Header: licensor name + point of contact, invoice number, invoice date, payment terms, currency, total amount, current status.
- Line items table: territory, plan, amount, line status (Paid/Unpaid).
- Payment history table: payment date, amount, line item paid, processor, batch ID (if applicable) — append-only, chronological.
- Outstanding balance: total − sum(payments), shown prominently.
- Actions: "Pay this invoice" (jumps into the batch payment flow pre-filtered to this invoice's unpaid lines), "Mark as Blocked" (manual toggle with a required reason note).

### 4.4 Invoice Payments (Batch Payment Run)
- A queue/selection view: filterable list of all outstanding (unpaid) line items across invoices — by licensor, territory, plan, due date.
- Ops user checks the line items to pay (can span multiple invoices/licensors).
- Since lines are full-or-nothing, each checked line pays its full amount — no amount entry needed, just selection.
- Review step: summary of total $ amount, count of line items, count of invoices affected, grouped by currency.
- Submit → creates one Payment record per line item, all sharing a single batch_id, payment_date = today, processor = current user.
- After submission: affected invoices' statuses recompute automatically.

## 5. Status Logic

Invoice status is computed from its line items, except Blocked which is a manual override that takes precedence:

- **Blocked** — manually set by an ops user, regardless of underlying payment state.
- **Paid** — all line items have status Paid.
- **Partially Paid** — at least one line item Paid, at least one Unpaid.
- **Unpaid** — no line items Paid.

## 6. Key Assumptions

- Invoices are entered by ops staff (upload or manual) — no direct licensor self-service portal in this scope.
- Line items are paid in full only; no partial-dollar payment against a single territory/plan line.
- "Blocked" is manual only in this version — no automated block rules (e.g., missing tax docs) yet, though the field is structured to support that later.
- Currency conversion/FX is out of scope — invoices are tracked in their original currency as submitted.
- No approval workflow (e.g., multi-step sign-off before payment) — single ops user can create and pay invoices. Noted as a likely real-world gap, called out in rationale.

## 7. Visual Design Direction

Apple internal tooling aesthetic: light background, generous whitespace, SF Pro (or system default sans-serif as a stand-in), minimal chrome, data presented in clean tables with subtle borders rather than heavy grid lines, understated status badges (muted color fills, not saturated red/green alarm colors), left-nav or top-nav simple wayfinding between the four screens.

## 8. Out of Scope (this exercise)

- Real authentication/permissions
- FX conversion
- Multi-step approval workflows
- Automated invoice ingestion from licensor systems
- Notifications/reminders for upcoming due dates
