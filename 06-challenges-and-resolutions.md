# 6 · Key Challenges & How I Resolved Them

[← Back to index](./)

---

Each challenge below is written in **STAR** form (Situation · Task · Action · Result). Together they
show the range of the work: domain-correctness, financial integrity, security, build/release
engineering, and platform quirks.

---

## Challenge 1 — Money that must never drift

**Situation.** An ERP that prints GST invoices and keeps double-entry books cannot afford floating-point
rounding error. Even sub-paise drift accumulates into wrong invoices and books that don't reconcile —
an audit and compliance failure (logged up-front as risk **R1**).

**Task.** Guarantee exact money and quantity arithmetic across the entire stack.

**Action.** I locked a hard invariant: **money is stored as integer paise, quantity as integer
milli-kg — never floats** — and scaling to rupees/kg happens *only at display* via dedicated
formatters. The money/quantity conversion helpers and the finance-derivation logic (aging, trial-balance
sign, FY dates) were built as pure functions with **unit tests** (`money.test.ts` and friends — 51 cases
across 6 files); valuation (WAC/FIFO) and GST run as server-side functions. The integer rule was enforced
consistently and made part of every relevant Definition of Done.

**Result.** No float arithmetic touches money anywhere in the system; the money/quantity foundation is
unit-tested and tax/valuation run server-side on integers. This is the foundation the entire finance
module stands on.

---

## Challenge 2 — Two business models that must never contaminate each other

**Situation.** The company runs Purchase–Sale *and* Job-Work. Confusing them — valuing a customer's
steel as company stock, or claiming GST input credit on job-work material — is a real legal/tax
exposure (risk **R3**).

**Task.** Make it structurally impossible to mix the two.

**Action.** I made `order_type` **sacred**: an enum enforced at the database layer drives stock impact,
GST input eligibility, and billing. Job-work flows are built so customer material **never** enters
the company's stock-value ledger and **never** generates GST input; job-work dispatches bill **charges-only**
with an SAC code and no material line, on a separate gap-free invoice series.

**Result.** The separation is enforced by the schema and transaction logic, not by operator memory —
removing an entire category of compliance risk.

---

## Challenge 3 — The books were silently wrong (financial-integrity defect)

**Situation.** During the v1 finance audit I discovered a serious latent defect: the chart-of-accounts
balance column (`bal_paise`) was **never updated by any posted transaction**. Every sale, purchase, and
payment correctly inserted a journal entry — but the *reported* P&L and Balance Sheet were reading
**frozen seed numbers disconnected from the ledger.** The finance screens looked fine and were
completely untrustworthy. A second issue compounded it: account IDs were **hardcoded string literals**
scattered through the transaction functions (code drift).

**Task.** Make the reported books provably equal the ledger, and remove the hardcoded IDs — *before*
building any finance feature on top.

**Action.** A three-migration fix:
1. Introduced an `opening_paise` baseline and an **AFTER-INSERT trigger** on journal entries that
   adjusts both legs' balances by their normal side — so balances now move with every posting.
2. Added a `verify_ledger()` reconciliation function asserting the invariant
   **`bal_paise = opening_paise + Σ(journal movement)`**, returning `balanced: true/false`.
3. Added an `accounting_config` registry and refactored the transaction functions to **resolve account
   IDs from config** instead of hardcoded literals. Because journal entries are append-only, the
   AFTER-INSERT trigger is safe by construction.

**Result.** `verify_ledger()` returns **balanced** (confirmed on the live database). I then surfaced it
in a **Trial Balance "books-health" banner**, and built standalone P&L, Balance Sheet, and Aging on the
now-trustworthy ledger. This is the project's clearest example of **catching a correctness bug that the
UI was hiding, fixing the foundation, then building on it** — not papering over it.

---

## Challenge 4 — Gap-free invoice numbering under concurrency

**Situation.** GST invoice series must be **gap-free** and correctly partitioned (sale `ISI/...` vs
job-work `JW/...`), even when dispatches are created close together.

**Task.** Guarantee no gaps and no duplicates per series.

**Action.** Invoice numbers are allocated inside the atomic dispatch transaction under a **Postgres
advisory transaction lock**, so concurrent dispatches serialize on number allocation. Job-work and sale
share the allocation logic but draw from **separate series**, and job-work "trucks" are excluded from
goods-dispatch counts.

**Result.** Gap-free numbering per series, verifiable by the DoD ("invoice numbers gapless per series").

---

## Challenge 5 — The CI pipeline was silently not building the APK

**Situation.** The release artefact for this project is an Android APK built in CI. I found that CI had
been **effectively dead for weeks**: runs had collapsed from ~3 minutes to ~10 seconds, meaning the web
job was failing instantly and the Android/APK job (which depends on it) was being **skipped — no APK had
been produced in CI at all.** Locally everything looked green because a `--legacy-peer-deps` flag had
been masking the problem.

**Task.** Diagnose why a "green-looking" local build was a broken CI, and restore a real APK artefact.

**Action.** Root cause: a **dead dependency** (`@capacitor/preferences@8`, whose only consumer had been
deleted) peer-required a newer Capacitor core than the pinned version, so a clean `npm ci` in CI failed
with `ERESOLVE`. The local legacy-peer-deps flag hid it. I **removed the dead dependency**, regenerated
the lockfile, and re-synced the native project so the plugin was dropped cleanly.

**Result.** `npm ci` runs clean; CI went green and produced its **first APK artefact in weeks**
(confirmed on the Actions dashboard). Lesson captured: *a local build that needs a "make it work" flag
is a warning sign — trust the clean install.*

---

## Challenge 6 — Free-tier and offline platform quirks

**Situation.** Building free-first and mobile surfaced a cluster of platform-specific bugs that don't
appear in a normal cloud web app.

**Task.** Make the app robust on a real phone over LAN/4G, offline-capable for static assets, and
secure under a strict content policy.

**Action — three representative fixes:**
- **Fonts:** the app loaded fonts from a CDN that the strict **Content-Security-Policy blocked** — so it
  silently fell back to system fonts on web and rendered *dead* offline on Android. I **self-hosted**
  the fonts as bundled assets, removing the CDN dependency entirely.
- **A "dead button" on mobile:** a Save action worked on desktop but threw on the phone. Root cause:
  the code called `crypto.randomUUID()`, which is **undefined outside a secure context** (LAN over plain
  HTTP on a phone). I added a non-crypto fallback so the action completes.
- **OCR that's "assist, not trust":** for bill scanning, I used **on-device OCR** (native ML Kit /
  Tesseract.js on web) so no paid OCR service was needed, with a CSP carve-out for the OCR core and a
  **template-aware parser** (e-invoice + Tally layouts + a generic fallback). Every parsed field is
  human-confirmed before save, and an OCR failure never blocks manual entry.

**Result.** Fonts render correctly on web and offline; mobile save works over LAN-HTTP; bill scanning
pre-fills inward bills accurately at zero service cost — all while staying inside the security policy.

---

## Challenge 7 — Authentication that kept looping

**Situation.** Early on, the email one-time-passcode login **looped** — users couldn't get in.

**Task.** Find the real root cause rather than patching symptoms.

**Action.** I traced it to **backend/dashboard configuration**, not application code: the email template
link type, the redirect-URL allow-list, and the OTP-length setting. I made the OTP handling flexible,
added resend with a cooldown, documented the exact dashboard configuration in a setup runbook, and later
moved the PIN to a **backend bcrypt-hashed store** (replacing a weak device-only scheme) with profile
capture on first sign-up.

**Result.** The end-to-end flow (email → code → set-up → PIN → home) was confirmed working on the
owners' devices. Key lesson: *for auth, the bug is often in configuration — read the whole flow before
touching code.*

---

## Challenge 8 — Securing a client-shipped app

**Situation.** A mobile app ships its bundle to users' devices; anything secret in it is exposed (risk
**R9**). Finance data must also be invisible to the operations role.

**Task.** Enforce least privilege and keep all secrets server-side.

**Action.** Only the safe public (anon) key ships in the client; **Row-Level Security** (default-deny +
per-role policies) is the authorization backbone, with finance tables **explicitly denied to the ops
role** — enforced in the database *and* re-checked in the UI for defence-in-depth. Sensitive functions
are `SECURITY DEFINER` with a pinned `search_path` and least-privilege grants; the financial and stock
**ledgers are append-only** (UPDATE/DELETE blocked by a DB trigger), and an admin-only `audit_log` table
is in place for recording mutations. When a key was suspected exposed, I **rotated it and verified the
old one was dead** (REST 401), recording the change in the out-of-git registry.

**Result.** An ops-role query against finance tables returns **zero rows**; secrets stay server-side;
the ledgers are tamper-evident (append-only); key rotation is a documented, verified procedure.

---

## What these challenges have in common

Across all eight, the same operating pattern shows up: **find the real root cause (not the symptom),
fix it at the right layer (often the database or config, not the UI), prove it with a test or a
verifiable check, and record it** so it can be rolled back or repeated. That discipline is what let a
solo, AI-assisted build hold an enterprise-grade correctness bar.

---

[← Previous: Prioritization & Stakeholders](05-prioritization-and-stakeholders.md) · [Next: Outcome & Business Impact →](07-outcome-and-impact.md)
