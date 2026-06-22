# Product Requirements Document — Steel ERP

> **Status:** Shipped (beyond MVP) · **Type:** Internal mobile ERP · **Owner:** Solo PM + Developer
> 🔒 Business identifiers anonymized for public sharing.

[← Back to Product Documents](README.md)

---

## 1. Overview

Steel ERP is an internal, Android-first mobile ERP for a **steel round-bar processing MSME** in
Gujarat, India. It digitizes the company's complete order-to-cash operation — sales orders,
procurement, production on draw benches, dispatch, GST invoicing, and double-entry finance — replacing
paper registers, spreadsheets, and WhatsApp coordination.

## 2. Problem statement

The business runs two business models (Purchase–Sale and Job-Work) on manual tooling, causing:

- No single source of truth across orders, stock, production, and finance.
- Error-prone money/GST maths, especially the sale-vs-job-work distinction (a compliance risk).
- No audit trail and no real-time view of stock, receivables, or production.

## 3. Goals & success metrics

| Goal | Success metric |
|------|----------------|
| Digitize the full order-to-cash chain | All core modules live on a real backend (achieved: 28 screens) |
| Exact financial maths | Integer-paise arithmetic; valuation tests pass to the paisa |
| Balanced, auditable books | `verify_ledger()` returns balanced; append-only ledgers (trigger-blocked) |
| Correct GST treatment | Automated intra/inter split; job-work billed charges-only (SAC) |
| Zero run cost | ₹0/month, no payment card attached |
| Installable release | Signed-APK CI pipeline |

## 4. Non-goals (explicit scope boundaries)

- **Not** a public multi-tenant SaaS — single company, single GSTIN, internal users only.
- **No** App Store / Play Store distribution — sideloaded signed APK.
- **No** full offline-sync at MVP (graceful degradation on poor 4G instead).
- **No** live GSTN/NIC e-invoice/e-way API integration at MVP — records the data, exports returns.
- iOS is "nice to have", deferred (carries an unavoidable Apple Developer cost).

## 5. Personas

| Persona | Device | Scope | Role |
|---------|--------|-------|------|
| **Operations owner** | Android phone (shop floor) | Inwards, job cards, draw benches, dies & stores, dispatch, scrap | `ops` (finance denied) |
| **Finance owner** | PC + phone | GST, bills, job-work charges, payments, banking, vouchers, analytics | `finance` |
| **Admin** (either owner) | Any | User mgmt, master data, settings, audit | `admin` |
| **Worker / Accountant** (future) | — | Restricted views | reserved |

## 6. Scope — feature inventory (F1–F22)

| # | Feature | Module | MVP? |
|---|---------|--------|------|
| F1 | PIN/password login, signup, forgot-PIN | Auth | ✅ |
| F2 | Role-based access (Ops/Finance/Admin) | Auth | ✅ |
| F3 | Home dashboard: KPIs, alerts, module grid, activity | Home | ✅ |
| F4 | Sales orders + fulfillment plan + srcType + flow callout + doc chain | Orders | ✅ |
| F5 | Supplier POs + multi-truck receipts | Orders | ✅ |
| F6 | Inward bills (Purchase/Jobwork/Trading-FG), multi-item, +OCR-assist | Inwards | ✅ |
| F7 | Inventory: lots, valuation, manual adjust, scrap, pickling loss, long-term flag | Inventory | ✅ |
| F8 | Job cards: matLog/outLog, per-drawbench queue, die auto-match | Ops | ✅ |
| F9 | Draw benches + utilisation | Ops | ✅ |
| F10 | Dies & Stores (dies + consumables + spare bills) — add/remove/adjust/rework | Dies & Stores | ✅ |
| F11 | Dispatch (sale + job-work) + multi-truck + shared invoice sequence | Dispatch | ✅ |
| F12 | GST returns + E-Way bill no./date capture (manual + OCR) | Dispatch/GST | ✅ |
| F13 | Finance: COA, journal, vouchers, ledgers, payments recv/made, bank match | Finance | ✅ |
| F14 | E-invoice payload/record (IRN/ack) | Finance | ⬜ v2 (parked) |
| F15 | Reports + PDF/Excel export | Reports | ✅ core |
| F16 | Universal search | Search | ✅ |
| F17 | Parties (suppliers/buyers) master + detail | Parties | ✅ |
| F18 | Notifications / alerts | Home | ✅ |
| F19 | Settings, theme (light/dark), profile | Settings | ✅ |
| F20 | Workforce + attendance | Workforce | ⬜ v2 (parked) |
| F21 | Analytics dashboard | Analytics | ✅ (shipped post-MVP) |
| F22 | Append-only ledgers + `audit_log` table (admin-only RLS; per-mutation trigger pending) | Cross-cutting | ◑ |

**MVP cut line:** F1–F13, core F15, F16–F19, F22. Deferred: F14, F20, and long-tail reports.
*(Because the UI was frozen, every screen ships; "deferred" = backend wiring stubbed for v2.)*

## 7. Key functional requirements

- **Fulfillment:** each sales-order line resolves to one srcType — `stock` · `produce` · `procure` ·
  `trade` · `jobwork` — or a partial split; the app shows the resulting flow scenario.
- **Multi-truck:** one Supplier PO arrives over N inward trucks; one sales order ships over N dispatch
  trucks; cumulative received/dispatched tracked.
- **Valuation:** every stock movement carries a cost lot; consumption applies WAC (default) or FIFO;
  closing value = sum of remaining lots; pickling loss (~3%) moves raw → scrap.
- **GST:** intra-state (seller state 24) → CGST+SGST; inter-state → IGST; job-work → SAC + charges only.
- **Invoice series:** gap-free per series (sale vs job-work), allocated under a transaction lock.
- **Finance:** every dispatch/inward/payment posts a balanced double-entry voucher; books reconcile via
  `verify_ledger()`.
- **Dies:** identified by internal diameter; rework enlarges the bore to the next standard diameter;
  track stock, reorder level, rework count.

## 8. Non-functional requirements

| Category | Requirement |
|----------|-------------|
| **Correctness** | Money = integer paise; quantity = integer milli-kg; never floats |
| **Security** | RLS default-deny + per-role; finance denied to ops; secrets server-side; append-only ledgers (trigger-blocked) |
| **Reliability** | Atomic transactions; idempotent retries on flaky 4G; graceful degradation |
| **Cost** | ₹0/month; no payment card on any cloud account |
| **Quality** | lint 0 · strict TypeScript · tests green · build green at every commit |
| **Platform** | Android-first; installable signed APK; no app store |

## 9. Release milestones

M0 Foundations → M1 Prototype migration → M2 Auth & RBAC → M3 Core data → M4 Orders & inwards →
M5 Stock & valuation → M6 Operations → M7 Dispatch & GST → M8 Finance & reports → M9 Hardening →
M10 Distribution. *(See [Requirements → User Stories](../04-requirements-to-user-stories.md).)*

## 10. Risks

See the [BRD risk register](BRD.md#7-risk-register-r1r12) (R1–R12) — float drift, job-work/purchase GST
confusion, valuation complexity, free-tier pauses, secrets-in-client, and more, each with a mitigation.

---

[← Back to Product Documents](README.md)
