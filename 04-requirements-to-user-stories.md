# 4 · From Business Requirements to Development Tasks & User Stories

[← Back to index](README.md)

---

## 4.1 The traceability chain

Every line of code traces back to a business reason through a four-link chain. This is the backbone of
how requirements became shippable work:

```
BUSINESS RULE  →  FEATURE (F1–F22)  →  MILESTONE (M0–M10)  →  TASK (with Definition of Done)
```

- **Business rule** — a fact about how the company must operate (e.g. "job-work never enters our
  stock value").
- **Feature** — a user-facing capability in the feature inventory (F1–F22).
- **Milestone** — a coherent delivery slice (M0 Foundations … M10 Distribution).
- **Task** — an executable unit of work with an explicit, testable **Definition of Done (DoD)**.

Because each link points back to the one before it, I could always answer *"why are we building this?"*
and *"how do we know it's done?"* — for any task.

## 4.2 The feature inventory (F1–F22)

Business rules were first consolidated into a numbered feature inventory, each tagged MVP or v2 and
traced to its prototype source. A sample:

| # | Feature | Module | MVP? |
|---|---------|--------|------|
| F1 | PIN/password login, signup, forgot-PIN | Auth | ✅ |
| F2 | Role-based access (Ops / Finance / Admin) | Auth | ✅ |
| F4 | Sales orders + fulfillment plan + srcType + doc chain | Orders | ✅ |
| F6 | Inward bills (Purchase/Jobwork/Trading-FG), multi-item, OCR-assist | Inwards | ✅ |
| F7 | Inventory: lots, valuation, scrap, pickling loss, long-term flag | Inventory | ✅ |
| F8 | Job cards: matLog/outLog, per-drawbench queue, die auto-match | Ops | ✅ |
| F11 | Dispatch (sale + job-work) + multi-truck + shared invoice sequence | Dispatch | ✅ |
| F13 | Finance: COA, journal, vouchers, ledgers, payments | Finance | ✅ |
| F14 | E-invoice IRN/IRP live | Finance | ⬜ v2 |
| F20 | Workforce + attendance | Workforce | ⬜ v2 |
| F21 | Analytics dashboard | Analytics | ⬜ v2 → **shipped** |

*(Full F1–F22 list in the [PRD](08-product-docs/PRD.md).)*

## 4.3 The milestone roadmap (M0 → M10)

Features were sequenced into milestones so that each milestone delivered something coherent and
testable, and later milestones depended only on earlier ones:

| Milestone | Theme | Delivers |
|-----------|-------|----------|
| **M0** | Foundations | Repo, tooling, Supabase, Capacitor shell, CI |
| **M1** | Prototype migration | All 28 screens ported, modular + typed, on mock data (UI parity) |
| **M2** | Auth & RBAC | Auth flows + roles + Row-Level Security |
| **M3** | Core data + masters | Full schema migrations + masters CRUD |
| **M4** | Orders & inwards | Sales orders, fulfillment plan, multi-truck inward, OCR-assist |
| **M5** | Stock & valuation | Cost-lot ledger, FIFO/WAC engine, scrap, pickling loss |
| **M6** | Operations | Job cards, draw benches, dies & stores (with rework) |
| **M7** | Dispatch & GST | Dispatch, invoice series, GST split, E-Way, payments |
| **M8** | Finance & reports | COA, journal, vouchers, core reports + export |
| **M9** | Hardening | Security, performance, QA |
| **M10** | Distribution | Signed APK release |

## 4.4 The Definition of Done discipline

Every task carried a **Definition of Done** — a concrete, verifiable condition, not "looks good". This
is what kept "done" honest across a solo build. Examples straight from the task list:

| Task | Definition of Done |
|------|--------------------|
| Multi-truck inward (`receive-inward`) | *"2 inwards on one Supplier PO → 100% received, balance 0; multi-item bill renders."* |
| Valuation engine (FIFO/WAC) | *"Golden-number unit tests (FIFO + WAC, multi-inward) pass to the paisa."* |
| Dispatch + invoice series | *"Invoice numbers gapless per series; job-work invoice has SAC + charges, no material line."* |
| GST + E-Way | *"Inter → IGST, intra → split; ₹49k no prompt, ₹51k prompts."* |
| Job cards | *"Production moves raw → finished goods; a job-work card doesn't touch the company's stock value."* |
| Double-entry posting | *"Every dispatch/inward/payment posts a balanced entry (Σ debit = Σ credit)."* |
| Roles + RLS | *"An Ops query on `payments` returns 0 rows; Finance succeeds."* |

A DoD that names a number (`balance 0`, `gapless`, `Σdr = Σcr`, `0 rows`) can be turned directly into a
test or a manual check — which is exactly what happened.

## 4.5 Worked example: "one Supplier PO, many trucks"

Here is the full chain for one real requirement, end to end:

**1 · Business rule** — *"A supplier may deliver one purchase order across several trucks on different
days. We need to track cumulative received quantity and know when the PO is fully received."*

**2 · Feature** — F5 (Supplier POs + multi-truck receipts) + F6 (inward bills).

**3 · Milestone** — M4 (Orders & inwards).

**4 · Task** — *"`receive-inward` Edge Function: atomically create the bill + items + cost lot + stock
ledger entry; support multiple inwards per Supplier PO; show a receipt sequence."*

**5 · User story** —
> *As the Operations owner, I want to record each truck's delivery against a Supplier PO, so that I can
> see how much of the order has arrived and how much is still pending.*

**6 · Acceptance criteria (the DoD, made testable)** —
- Given a PO for 1000 kg, when I receive 600 kg then 400 kg, the PO shows **100% received, balance 0**.
- Each receipt creates its own cost lot and an **append-only** stock-ledger entry.
- A multi-item bill renders all line items.
- The whole receipt is **atomic** — a partial failure writes nothing.

**7 · Verification** — covered by the DoD check above plus the integrity rule that *ledger sum equals
current stock*.

## 4.6 Worked example: "a sale and a job-work charge on the same order"

**Business rule** — *"A single order can ship goods (taxable sale) AND bill job-work conversion
(SAC, charges only). The two must not contaminate each other."*

→ **Feature** F11 (dispatch) → **Milestone** M7 → **Task**: *"`create-dispatch` posts atomic
stock-out + gap-free invoice number per series + GST + voucher; job-work = charges-only + SAC;
sequence excludes job-work from goods-truck counts."*

→ **User story**: *As the Finance owner, I want job-work dispatches billed as charges-only with an SAC
code, so that I never accidentally treat a customer's material as a taxable goods sale.*

→ **Acceptance criteria**: job-work invoice has **SAC + charges, no material line**; goods dispatches
get a `ISI/...` invoice number, job-work a separate `JW/...` series, **both gap-free**; the sale and
job-work appear as distinct, correctly-taxed lines.

## 4.7 Keeping the backlog alive

The roadmap wasn't frozen. As the build progressed, a **v1 closeout audit** produced a working
roadmap of 16 remaining features, each with a status and a route, refined as items shipped. This is
how the backlog stayed an accurate planning instrument rather than a stale wish-list — and it's the
basis for the prioritization decisions in the next document.

> The full epics-and-stories backlog (with acceptance criteria) lives in
> [User Stories](08-product-docs/USER-STORIES.md).

---

[← Previous: Requirements Gathering](03-requirements-gathering.md) · [Next: Prioritization & Stakeholders →](05-prioritization-and-stakeholders.md)
