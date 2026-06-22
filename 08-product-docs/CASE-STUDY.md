# Case Study — Building a Steel MSME's ERP, Solo, Beyond MVP

> A product-and-engineering case study. 🔒 Business identifiers anonymized for public sharing.

[← Back to Product Documents](README.md)

---

## TL;DR

I took a **steel round-bar processing MSME** in Gujarat from running on paper, spreadsheets, and
WhatsApp to a **beyond-MVP, mobile-first ERP** with exact money/GST maths, balanced double-entry books,
and a signed-APK release pipeline — built **solo as both Product Manager and Developer**, directing an
AI coding agent, at **₹0/month** running cost.

**28 screens · 24 DB migrations · 19 Edge Functions · 51 tests green · balanced ledger · ₹0/month.**

---

## Situation

A small steel manufacturer runs **two business models in one shop**: it buys raw bars, draws them into
finished bright bars and sells them (Purchase–Sale), and it also processes customers' own material for a
fee (Job-Work). Everything — orders, stock, job cards, dispatch, GST, payments — lived on paper and
WhatsApp.

That created the classic MSME risk surface: no single source of truth, drifting spreadsheets,
error-prone GST maths, and a genuine compliance hazard if a job-work job is ever mistaken for a taxable
goods sale. Off-the-shelf ERPs are expensive, over-featured, and don't model the company's specific
draw-bench + job-work + multi-truck reality.

## Task

Deliver a **purpose-built, internal, Android-first ERP** that:

- runs the entire order-to-cash chain with **correct money and GST**,
- keeps **balanced, auditable, double-entry books**,
- enforces the **sale-vs-job-work** separation structurally,
- costs **₹0/month** with no vendor lock-in, and
- ships as an **installable signed APK** —

all built and maintained by **one person**.

## Action

**As Product Manager**, I anchored requirements on a **frozen interactive prototype the owners had
already approved** — making it the contract ("when the doc and the prototype disagree, the prototype
wins"). I reverse-engineered the real data model and business rules from it, logged the unknowns as an
open-questions register tied to the modules they blocked, recorded assumptions explicitly, and hunted
down the edge cases (partial fulfillment, multi-truck, job-work scrap ownership, die rework) that break
ERPs in production. I built a feature inventory (F1–F22), a milestone roadmap (M0–M10) with a
**Definition of Done** per task, an MVP cut line, a **free-first cost mandate**, and a **parked-feature
decision log**.

**As Developer / Tech Lead**, I chose a free-first stack (Capacitor + React + TypeScript strict + Vite
+ Supabase), designed a 35-table relational schema with **append-only ledgers**, locked the core
invariants (**money in integer paise, quantity in integer milli-kg, never floats**), and enforced
security with **Row-Level Security** (finance denied to the ops role), server-side secrets, and an
append-only ledgers. I implemented atomic transaction functions and 19 Edge Functions for GST,
valuation, vouchers, and reporting, and kept **four quality gates green at every commit**.

**Directing an AI coding agent** as a force-multiplier, I owned the spec, the decisions, the reviews,
and the integration — the agent accelerated execution under my review.

**The defining moment** was a finance-integrity audit that revealed the reported P&L and Balance Sheet
were reading **frozen seed numbers disconnected from the ledger** — the screens looked fine and were
untrustworthy. I fixed it at the foundation: a balance-sync trigger, a `verify_ledger()` reconciliation
check, and a config registry replacing hardcoded account IDs — *then* built Trial Balance, P&L, Balance
Sheet, and Aging on the now-verified ledger.

## Result

- **Beyond MVP, feature-complete for the core business** — 28 screens live on a real backend, 24
  migrations applied, 19 Edge Functions deployed.
- **Financially correct** — `verify_ledger()` returns **balanced**; money is integer-paise throughout;
  GST is split automatically and job-work is billed charges-only.
- **Engineered to a bar** — **51 tests** passing; lint 0 / strict TypeScript / build green at every
  commit; CI restored to green and producing a real APK artefact.
- **Secure & tamper-evident** — RLS, server-side secrets, append-only financial & stock ledgers, verified
  key rotation.
- **₹0/month** with full ownership and a self-hostable stack (no lock-in), and a **signing-ready release
  pipeline**.

## What it demonstrates

- **Product judgement** — anchoring on a validated artefact, scoping ruthlessly, right-sizing process to
  a 2-user app, and saying "not now" with recorded reasons.
- **Technical depth** — financial correctness, security by default, atomic transactions, and CI/release
  discipline.
- **A modern operating model** — one person + an AI agent + specs, reviews, and gates, producing
  software that would normally need a team — without compromising correctness.

> Full detail in the [PRD](PRD.md), [BRD](BRD.md), [User Stories](USER-STORIES.md), and the
> [Challenges](../06-challenges-and-resolutions.md) write-ups.

---

[← Back to Product Documents](README.md)
