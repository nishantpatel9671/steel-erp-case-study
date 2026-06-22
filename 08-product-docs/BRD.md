# Business Requirements Document — Steel ERP

> 🔒 Business identifiers anonymized for public sharing.

[← Back to Product Documents](./)

---

## 1. Business context

A **steel round-bar processing MSME** in Gujarat, India buys raw steel bars, processes them on **draw
benches** into finished bright bars, and sells them — and also performs **job-work** (processing a
customer's own material for a conversion fee). The business runs on paper, spreadsheets, and WhatsApp,
with no integrated system, no reliable books, and no audit trail.

The business needs a system that captures the **complete order-to-cash workflow** with correct money
and GST, owned by the company, at minimal cost.

## 2. The two business models

| Model | Description | Billing | Stock & GST |
|-------|-------------|---------|-------------|
| **Purchase–Sale** | Buy raw → process → sell finished goods | Full GST invoice | Material enters stock value; GST input credit applies |
| **Job-Work** | Customer's material in → process → return | Charges-only (SAC) | Customer material never enters stock value; no GST input on it |

These must be kept strictly separate — mixing them creates incorrect inventory valuation and
GST-input/compliance exposure.

## 3. Business objectives

1. One source of truth for orders, procurement, stock, production, dispatch, and finance.
2. Exact money and GST — correct invoices, correct tax treatment, books that balance.
3. Auditability — every transaction attributable and reconcilable.
4. Real-time operational visibility (stock, receivables/payables, production queues, alerts).
5. Mobile usability for the shop floor.
6. Zero ongoing cost and full ownership of the system.

## 4. Business rules (the rules the software must enforce)

1. **Order/JC type is sacred** — every inward is Purchase | Jobwork | Trading-FG; every job card is
   Purchase-Sale | Jobwork. It drives stock impact, GST input, and billing.
2. **Job-work never touches stock value or GST input** — customer material is never valued as company
   stock; job-work bills conversion charges only (SAC), no material line.
3. **Money in integer paise, quantity in integer milli-kg** — never floats; round only on display.
4. **GST split** — intra-state (seller state 24) → CGST 9% + SGST 9%; inter-state → IGST 18%;
   auto-detected from buyer state.
5. **Gap-free invoice series** — separate series for sale vs job-work; no gaps, no duplicates.
6. **Valuation** — cost lots per movement; WAC (default) or FIFO consumption; pickling loss (~3%) moves
   raw → scrap.
7. **Double-entry** — every dispatch/inward/payment posts a balanced voucher (Σ debit = Σ credit).
8. **Multi-truck** — cumulative tracking of receipts against a Supplier PO and dispatches against a
   sales order.
9. **Dies by diameter** — rework enlarges the bore to the next standard diameter; track reorder + rework
   count.
10. **E-Way bill** — legally required above ₹50,000; the app records the E-Way bill no./date on
    dispatches & inward bills (manual entry or OCR-extracted). *(Threshold is not auto-enforced in-app.)*
11. **Audit** — financial & stock ledgers are append-only (UPDATE/DELETE blocked by a DB trigger); an
    admin-only `audit_log` table is provisioned for mutation logging.
12. **Access** — finance data restricted to the finance/admin roles; operations role denied.

## 5. Constraints

| Constraint | Detail |
|------------|--------|
| **Cost** | Free-first; ₹0/month; **no payment card** attached to any cloud account (eliminates surprise billing) |
| **Platform** | Android-first; sideloaded signed APK; no app store |
| **Team** | Solo builder (PM + developer), AI-assisted; no dedicated DevOps — architecture must be operable by one person |
| **Connectivity** | Online-first; must degrade gracefully on poor 4G |
| **Tenancy** | Single company, single GSTIN; no multi-branch |

## 6. Assumptions

1. Two real, trusted users now; light RBAC today but it must exist for future staff.
2. Single company / single GSTIN; no multi-branch.
3. Online-first acceptable for MVP; full offline-sync out of scope.
4. Android priority; iOS deferred (carries an unavoidable Apple cost).
5. WAC valuation by default (GST-safe), FIFO available.
6. Live GST e-filing / e-way / e-invoice APIs out of scope at MVP — record + export only.
7. OCR is "assist, not trust" — every field human-confirmed before save.

## 7. Risk register (R1–R12)

| # | Risk | Mitigation |
|---|------|------------|
| R1 | Money/qty as floats → valuation & GST drift | Integer paise + milli-kg; round only at display |
| R2 | Inventory valuation (FIFO/WAC, multi-inward, pickling loss) is the hardest module | Pure server-side functions on integer paise; money/qty conversion unit-tested; lock valuation method early |
| R3 | Job-work vs purchase confusion in financials | `order_type` enum at DB; separate ledgers; no GST input on job-work |
| R4 | Free-tier project pause after inactivity | Uptime ping + documented un-pause; self-host option |
| R5 | Surprise billing if a card is attached | Do not attach a card to any cloud account |
| R6 | Single-file prototype unmaintainable as a real app | Migrate to a modular Vite project; UI preserved exactly |
| R7 | iOS needs a Mac + paid Apple Developer | Ship Android first (free); defer iOS |
| R8 | Vendor lock-in to a managed backend | Pick open-source, self-hostable backends; keep SQL portable |
| R9 | Secrets in the client bundle | Only the safe anon key ships; all secrets server-side; RLS |
| R10 | Audit/compliance gaps | Append-only ledgers enforced by DB triggers; `audit_log` table with admin-only RLS (generic per-mutation trigger not yet wired) |
| R11 | Browser-CDN build (prototype) insecure/slow | Replaced by an ahead-of-time Vite build |
| R12 | Drifting from the agreed UI during the port | Port verbatim; the prototype is the acceptance test |

## 8. Success criteria

- The full order-to-cash chain runs on a real backend.
- Books balance (`verify_ledger()` = balanced); valuation tests pass to the paisa.
- GST is correctly split and job-work is billed charges-only.
- The app runs at ₹0/month and produces an installable signed APK.

---

[← Back to Product Documents](./)
