# User-Story Backlog — Steel ERP

> Epics → user stories with acceptance criteria, grouped by module. Acceptance criteria are derived
> from the original tasks' Definitions of Done, so each story is verifiable.
> 🔒 Business identifiers anonymized for public sharing.

[← Back to Product Documents](README.md)

**Roles:** **Ops** = Operations owner · **Fin** = Finance owner · **Admin** = either owner.

---

## Epic A — Authentication & Access

**A1.** *As a user, I want to sign in with my email one-time code and a personal PIN, so that access is
secure and tied to me.*
- ✓ Email OTP → first-time setup captures name + mobile → backend-hashed PIN.
- ✓ Wrong PIN fails; a valid session persists across refresh.

**A2.** *As Admin, I want each user to have a role (Ops/Finance/Admin), so that people only see what
they should.*
- ✓ An **Ops** query against finance tables returns **0 rows**; **Finance** succeeds.
- ✓ Role enforced in the database (RLS) **and** re-checked in the UI.

---

## Epic B — Sales Orders & Fulfillment

**B1.** *As Ops, I want to create a sales order and choose how each line is fulfilled, so that the system
plans the right path.*
- ✓ Each line resolves to a srcType (`stock`/`produce`/`procure`/`trade`/`jobwork`) or a partial split.
- ✓ A flow-scenario callout shows the resulting path; the fulfillment plan tally is correct.

**B2.** *As Ops, I want to see an order's full document chain, so that I can trace it end to end.*
- ✓ SO detail shows PO → Supplier PO → Inward → Job Card → Dispatch → Payment.

---

## Epic C — Procurement & Inward (multi-truck)

**C1.** *As Ops, I want to raise a Supplier PO and receive it across several trucks, so that partial
deliveries are tracked.*
- ✓ Receiving 600 kg then 400 kg against a 1000 kg PO shows **100% received, balance 0**.
- ✓ Each receipt creates a cost lot + an append-only stock-ledger entry; the whole receipt is atomic.

**C2.** *As Ops, I want to scan a supplier bill to pre-fill an inward, so that data entry is faster.*
- ✓ On-device OCR pre-fills at least supplier + amount from a sample bill.
- ✓ Every field is human-confirmed before save; OCR failure never blocks manual entry.

---

## Epic D — Inventory & Valuation

**D1.** *As Ops, I want stock valued by cost lots, so that closing value is accurate.*
- ✓ Ledger sum equals current stock; no negative availability.
- ✓ WAC (default) / FIFO valuation runs server-side; the money/quantity conversion it relies on is unit-tested **to the paisa**.

**D2.** *As Ops, I want to log pickling loss and scrap, so that raw losses are recorded correctly.*
- ✓ Pickling loss moves quantity raw → scrap exactly; scrap statuses are settable.

---

## Epic E — Production (Job Cards & Draw Benches)

**E1.** *As Ops, I want to run a job card on a draw bench with material and output logs, so that
production is tracked.*
- ✓ Production moves raw → finished goods with yield/scrap; a **job-work** card does **not** touch
  the company's stock value.
- ✓ Job cards queue per draw bench; a die is auto-matched by output diameter.

**E2.** *As Ops, I want to rework a die, so that a worn bore is enlarged to the next standard size.*
- ✓ A rework increments the rework count, bumps the diameter to the next standard, and logs the date.

---

## Epic F — Dispatch & GST (multi-truck)

**F1.** *As Ops, I want to dispatch an order across trucks with a correct GST invoice, so that shipping
and billing are right.*
- ✓ Invoice numbers are **gap-free per series**; inter-state → IGST, intra-state → CGST+SGST.
- ✓ The E-Way bill no./date can be recorded on the dispatch (manual entry or OCR-extracted).

**F2.** *As Fin, I want job-work dispatches billed as charges-only, so that customer material is never
treated as a taxable goods sale.*
- ✓ A job-work invoice has **SAC + charges, no material line**, on a separate series.
- ✓ Job-work "trucks" are excluded from goods-dispatch counts.

---

## Epic G — Finance (double-entry)

**G1.** *As Fin, I want every transaction to post a balanced double-entry voucher, so that the books are
always correct.*
- ✓ Every dispatch/inward/payment posts an entry with **Σ debit = Σ credit**.
- ✓ `verify_ledger()` returns **balanced**.

**G2.** *As Fin, I want a Trial Balance, P&L, and Balance Sheet derived from the ledger, so that
statements reflect reality, not stale numbers.*
- ✓ Trial Balance shows a books-health banner + equal Dr/Cr totals.
- ✓ P&L is period-filtered; the Balance Sheet balances at any as-of date.

**G3.** *As Fin, I want to record supplier and customer payments, so that outstanding balances update.*
- ✓ A payment reduces outstanding and is reflected in the party ledger and aging.

**G4.** *As Fin, I want a receivables/payables aging report, so that I can chase overdue amounts.*
- ✓ Buckets 0-30 / 31-60 / 61-90 / 90+ with totals + an oldest-first detail table + CSV export.
- ✓ Payables (which use restricted data) are visible to finance/admin only.

---

## Epic H — Reporting & Analytics

**H1.** *As Fin, I want reports I can export, so that I can share and file them.*
- ✓ Report totals reconcile to their tables; CSV/PDF export opens.

**H2.** *As an owner, I want trend charts, so that I can see the business at a glance.*
- ✓ A 6-month Sales-vs-Purchases trend and production-output charts on the dashboard.

---

## Epic I — Cross-cutting

**I1.** *As any user, I want universal search, so that I can find any order/bill/dispatch/party fast.*
**I2.** *As an owner, I want alerts for low stock and dies due for reorder, so that I act in time.*
**I3.** *As any user, I want a light/dark theme, so that the app is comfortable to use.*
**I4.** *As Admin, I want the books to be tamper-evident, so that financial records can't be quietly altered.*
- ✓ The financial & stock ledgers are **append-only** — UPDATE/DELETE are blocked by a DB trigger.
- ◑ An admin-only `audit_log` table is provisioned for mutation logging (generic per-mutation trigger not yet wired).

---

## Parked (recorded, not yet built)

- **Workforce + attendance** — backend hook exists; UI wiring deferred.
- **Bank reconciliation (BRS)** — needs a bank-statement table + reconcile function.
- **Live e-invoice (IRN/IRP)** — externally gated on GSTN credentials; data shape already recorded.

---

[← Back to Product Documents](README.md)
