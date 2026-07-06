# Invoice & Payment Management Prototype

A UI prototype for an internal accounts-payable tool used by a music publishing operations team to manage licensor invoices — creation, territory/plan-level line items, and full or partial payment tracking.

**Design rationale / PRD:** [docs/PRD.md](./docs/PRD.md)

## The scenario

Apple Music receives invoices from music rights holders (call them licensors) which must be paid fully or partially. These invoices may be multi territorial and may contain one or more plans of Apple Music Service. Examples of plans - Individual, Family, Student etc.

## Task

Design a UI prototype for an internal invoice and payment management tool used by the Apple Music Publishing Operations team. Your prototype should cover:
- Creating and viewing invoices (with territory and plan details)
- Recording full or partial payments against an invoice
- Tracking outstanding balances and payment history per invoice.

## Key design decisions

- **Payments attach to line items, not invoice headers.** Since a licensor invoice can span multiple territories and plans, and payments can be partial *by* territory or plan, the payment record references the specific line item it settles rather than a lump amount against the invoice. Invoice-level status (Unpaid / Partially Paid / Paid) is computed as a rollup of its line items' paid state.
- **Line items are paid in full only.** No partial-dollar payment within a single territory/plan line — simplifies both the data model and the payment UI (selection, not amount entry).
- **Blocked is a manual, accountable override.** An ops user can block an invoice regardless of its payment state (e.g., a licensor dispute), and is required to enter a reason. It's independent of the computed status logic.
- **Batch payments are a selection flow, not a form.** Since lines pay in full, processing a batch of payments is: filter outstanding line items across licensors/territories/plans, select the ones to pay, review a summary grouped by currency, confirm. Each line generates its own payment record, all sharing one batch ID.

## Assumptions

- This is accounts payable: Apple is the payer, licensors are the payees.
- Invoices are entered by ops staff via manual form or CSV upload — no licensor self-service portal in scope.
- No FX conversion; invoices are tracked in their original currency.
- No multi-step approval workflow before payment in this version.
- Auto-block rules (e.g., missing tax documentation) are a plausible future extension, not built here — blocking is manual only for now.

Full assumptions and data model are in [docs/PRD.md](./docs/PRD.md).

## Stack

Single-file HTML/CSS/vanilla JS.
