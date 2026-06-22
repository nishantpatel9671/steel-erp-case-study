# 3 · How I Gathered & Understood Requirements

[← Back to index](./)

---

## 3.1 The core method: "the prototype is the contract"

The single most important decision in requirements was treating a **finished, frozen interactive
prototype** as the authoritative specification, rather than starting from a blank requirements doc.

The owners and I had already converged on a complete clickable prototype — every screen, field, flow,
colour, and label — that they had reviewed and accepted. Instead of re-debating the UI, I **declared
the prototype the source of truth** and made one rule explicit:

> **When the written requirements and the prototype disagree about a screen or a field name, the
> prototype wins** — fix the document, not the prototype.

This flipped the usual risk. Most projects drift *away* from the agreed design during build; here the
prototype was the **acceptance test**. The job was reframed from "design an ERP" to "make this exact,
already-approved app *real* — back it with a database, authentication, and correct server-side maths."

**Why this worked:** for a 2-user internal tool where the stakeholders are the domain experts, a
prototype they can click is a far higher-fidelity requirements artefact than prose they have to
imagine. It eliminated an entire class of "that's not what I meant" rework.

## 3.2 Reverse-engineering requirements from the prototype

The prototype was a single large file containing **25 screen cases, ~25 in-memory data arrays, and
~80 logic functions**. I mined it systematically to extract the *real* requirements hiding inside the
demo:

- **Data dictionary** — every in-memory array became a candidate table; nested arrays (`items[]`,
  `matLog[]`, `outLog[]`, `priceHistory[]`, `docs[]`) revealed the **child tables** a normalized schema
  needs.
- **Logic inventory** — every `_create*`, `_jc*`, `_stock*`, `_post*` helper revealed a **business
  rule** that had to move server-side (e.g. invoice numbering, valuation, voucher posting).
- **Screen inventory** — 25 routed + 3 auth screens, each with its exact field names, became the screen
  backlog.
- **Flow map** — tracing how the demo navigated revealed the **golden path** (order → fulfillment →
  production → dispatch → payment) the real app had to support.

## 3.3 Pre-flight: the 5 business questions

Before writing any code, I locked down the foundational business facts. These five had to be answered
because they shape the entire data model and money logic:

1. **What is this app, exactly?** Two business models (Purchase–Sale + Job-Work), draw-bench-only shop
   floor, six fulfillment scenarios.
2. **Who uses it?** Operations and Finance owners, with an Admin capability and future restricted roles.
3. **What are the money & quantity rules?** Integer paise and integer milli-kg, never floats.
4. **What are the GST rules?** Seller is in Gujarat (state 24); intra-state → CGST+SGST, inter-state →
   IGST; job-work uses SAC + charges only.
5. **How is it distributed?** Internal Android APK, no app store, free-first.

## 3.4 The open-questions log (OQ01–OQ14)

I didn't pretend to know everything. I kept an explicit **open-questions register**, marking which
questions *blocked* which modules so the right things got answered at the right time:

| Open question | Blocks |
|---------------|--------|
| **OQ08** — Inventory valuation: FIFO or WAC? | Entire stock & valuation module |
| **OQ05** — Invoice number series format (separate sale vs job-work series?) | Dispatch / invoicing |
| **OQ06** — GST rates per material category + SAC for job-work | GST auto-fill |
| **OQ11/14** — Job-work scrap ownership (the processor's or the customer's?) | Job-work invoice + scrap ledger |
| **OQ04** — Material master completeness (types, grades, sizes, HSN) | Inwards, job cards |
| **OQ09/10** — Draw-bench and die masters | Machine mgmt, Dies & Stores |
| **OQ12/13** — GST state confirmation + min-stock thresholds | Tax split, alerts |
| **OQ01/02** — Backend choice + OCR scope | All infra / inwards effort |

This is deliberate requirements hygiene: **surface unknowns early, attach them to the work they gate,
and proceed on documented assumptions where safe.**

## 3.5 The assumptions register

Where an answer wasn't yet available, I proceeded on **explicit, written assumptions** (each one
correctable), rather than silent guesses. Examples:

- Two real, trusted users now; light RBAC today but it must exist for future staff.
- Single company / single GSTIN; no multi-branch.
- Online-first acceptable for MVP; full offline-sync out of scope, but graceful degradation required.
- Android is the priority; iOS is "nice to have" and carries an unavoidable Apple cost.
- WAC valuation by default (simplest, GST-safe), FIFO available — final method per OQ08.

Documenting assumptions did two things: it let the build start without waiting on every answer, and it
gave stakeholders a concrete list to confirm or correct.

## 3.6 Catching the hidden requirements

The most valuable part of understanding requirements was finding the **edge cases the demo implied but
never said out loud**. These are exactly the things that break an ERP in production:

- A single order line can be **part ex-stock + part produced** (partial fulfillment).
- **Multiple inward trucks** can arrive against one Supplier PO before it's marked "received".
- One order can carry **both a sale dispatch and a job-work charge dispatch** — valuation must exclude
  job-work "trucks" from goods counts.
- **Trading-FG sits in the same finished-goods bucket** as own production — only the inward type
  distinguishes it, so traceability needs a `sourceInw` link.
- **Dies have finite life** — each rework enlarges the bore to the next standard diameter; needs
  run/rework counting and reorder alerts.
- **Pickling loss** (~3%) reduces raw stock and creates scrap.
- **E-Way bill only required above ₹50,000** — prompt conditionally.
- **Idempotency on retries** over flaky 4G, and explicit empty/zero states everywhere.

## 3.7 Translating domain into enforceable rules

Understanding requirements, for this domain, meant turning fuzzy business language into **rules the
software can enforce**. The key translations:

| Business reality | Enforceable rule |
|------------------|------------------|
| "Job-work isn't our material" | `order_type` enum at the DB; job-work never touches stock value or GST input |
| "Money must be exact" | Integer paise / milli-kg; round only at display |
| "Invoice numbers can't have gaps" | Gap-free series per type under an advisory transaction lock |
| "The books must balance" | Every transaction posts a balanced double-entry voucher; a `verify_ledger()` check |
| "Finance is private" | RLS denies finance tables to the ops role, enforced in the DB *and* the UI |
| "Nothing can be quietly edited" | `stock_ledger` and `journal_entries` are append-only via triggers |

## 3.8 What I'd highlight

Good requirements work here wasn't a 100-page spec. It was: **anchor on an artefact stakeholders had
already validated, mine it rigorously for the implied data model and rules, log the unknowns against
the work they block, proceed on explicit assumptions, and hunt down the edge cases before they became
production incidents.**

---

[← Previous: My Role](02-role-and-responsibilities.md) · [Next: Requirements → User Stories →](04-requirements-to-user-stories.md)
