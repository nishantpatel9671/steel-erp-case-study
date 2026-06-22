# 2 · My Role & Key Responsibilities

[← Back to index](README.md)

---

## 2.1 Summary

I owned this project **end-to-end as a solo builder wearing two hats**: **Product Manager** and
**Developer / Tech Lead**. There was no separate design team, no separate backend team, and no DevOps
function — I was responsible for everything from the first discovery conversation to a signed,
installable release candidate.

To deliver at that scope alone, I used an **AI coding agent (Claude Code) as a force-multiplier**. The
important distinction for how to read this document:

> **I owned the thinking; the agent accelerated the typing.** I authored the requirements, made every
> architecture and prioritization decision, defined the invariants the system must never break,
> reviewed and integrated every change, and gated every release. The AI executed against my spec under
> my review — the way a senior engineer directs and reviews a fast junior team.

This is a deliberate, modern operating model: **spec-driven, AI-executed, human-governed.** It let one
person ship enterprise-shaped software (double-entry finance, RLS security, atomic transactions, CI)
without cutting corners on correctness.

---

## 2.2 Product Manager responsibilities

| Responsibility | What I actually did |
|----------------|---------------------|
| **Discovery & domain learning** | Learned the steel round-bar manufacturing domain — draw benches, dies-by-diameter, pickling loss, scrap ownership, multi-truck logistics — and the two business models (Purchase–Sale vs Job-Work). |
| **Requirements definition** | Treated a finished interactive prototype as the "contract", then derived the real requirements, assumptions, and an open-questions log from it (see [Requirements Gathering](03-requirements-gathering.md)). |
| **PRD / BRD authoring** | Wrote the product and business requirements documents: problem, goals, personas, scope/non-goals, the 22-feature inventory, business rules, and success metrics (see [Product Documents](08-product-docs/)). |
| **Roadmap & milestones** | Structured the build into a milestone plan (**M0 Foundations → M10 Distribution**) with explicit tasks and a **Definition of Done** per task. |
| **Prioritization** | Drew the MVP cut line, set a **free-first cost mandate**, sequenced the v1 feature build order, and maintained a **parked-feature decision log** with rationale (see [Prioritization](05-prioritization-and-stakeholders.md)). |
| **Stakeholder management** | Managed the two owner-stakeholders on an "approve-on-parity" model — they validate that screens/flows match what was agreed; I translate their feedback into scoped changes. |
| **Governance & KPIs** | Right-sized an enterprise TPM charter down to what a 2-user internal MSME app needs — KPI governance, an ADR register, a risk register (R1–R12), cost governance, and a production-readiness gate. |
| **Release management** | Owned the release gate (signed-APK pipeline), the rollback discipline (every session = a tagged rollback point), and the out-of-git change registry (DB migrations + dashboard settings). |

## 2.3 Developer / Tech Lead responsibilities

| Responsibility | What I actually did |
|----------------|---------------------|
| **Architecture** | Chose the stack (Capacitor + React + TypeScript strict + Vite + Supabase) and the rationale for each layer; designed the client (TanStack Query + Zustand + RHF/Zod) and server (Postgres + Edge Functions) split. |
| **Database design** | Designed a 35-table relational schema for an ERP/GST/double-entry domain, with enums, constraints, indexes, and **append-only ledgers** enforced by triggers. |
| **Money & correctness** | Locked the core invariants: **money in integer paise, quantity in integer milli-kg**, never floats; scale only on display. Built valuation (WAC/FIFO) and GST logic as pure, unit-tested functions. |
| **Security** | Designed **Row-Level Security** (default-deny, per-role policies), `SECURITY DEFINER` functions with pinned `search_path` and least-privilege grants, finance-table denial for the ops role, CSP, and an immutable audit log. |
| **Server logic** | Implemented atomic transaction functions (complete-job-card, receive-inward, create-dispatch, settle-payment) and 18 Edge Functions for GST, valuation, vouchers, and reporting. |
| **CI/CD** | Maintained a GitHub Actions pipeline (lint → type-check → test → build, with APK build gated to releases) and the signing config for a release APK. |
| **Testing & QA** | Built the test suite (**51 tests**), ran whole-product QA passes, and kept four quality gates green (lint 0 / strict TS / tests / build) at every commit. |
| **AI-agent orchestration** | Wrote the execution prompts, set the coding standards and lint rules the agent had to satisfy, reviewed every diff, caught regressions, and integrated the results — the human-in-the-loop that kept quality high. |

## 2.4 Evidence this was genuinely owned (not just generated)

Portfolio claims are cheap; here is the **traceable evidence** baked into the repository:

- **A risk register written *before* code** (R1–R12) that correctly predicted the hardest problems —
  float drift, job-work/purchase GST confusion, valuation complexity, free-tier pauses, secrets in the
  client — and the mitigation for each. (See [Challenges](06-challenges-and-resolutions.md).)
- **A technical changelog with rollback points** — every working session is committed as a named
  rollback point, with an "out-of-git registry" tracking the DB migrations and dashboard settings that
  `git revert` cannot undo. This is release-management discipline, not vibe-coding.
- **A QA report** documenting a real bug I caught and fixed (a dispatch-validation field mismatch that
  was rejecting every dispatch), plus the extraction of pure logic into tested modules.
- **A self-imposed invariants list** ("NEVER violate") that the architecture enforces — money type,
  GST rules, RLS denial, append-only ledgers — proving the decisions were principled and consistent.
- **A governance layer** (Phase 11) mapping an enterprise PM charter onto a 2-user app and explicitly
  marking what was *deferred* — evidence of judgement about right-sizing process to context.

## 2.5 The operating model, honestly

I think the most interesting thing about this project is the operating model itself: **one person, an
AI agent, and a discipline of specs + reviews + gates** producing software that would normally need a
small team. The skill on display is not "I typed every line" — it's **knowing what to build, deciding
how it should be built, enforcing correctness, and reviewing everything that came back.** That is
exactly the blend of product and technical leadership the two roles in this document describe.

---

[← Previous: Product Overview](01-product-overview.md) · [Next: Requirements Gathering →](03-requirements-gathering.md)
