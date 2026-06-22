# 5 · Prioritization & Stakeholder Management

[← Back to index](README.md)

---

## 5.1 The prioritization philosophy

With a single builder and two stakeholders, prioritization had to be ruthless and explicit. Three
forces drove every decision:

1. **Correctness before breadth** — the modules that, if wrong, cause real money/tax/legal damage
   (finance, GST, valuation) were built to a higher bar than convenience features.
2. **Cost as a first-class constraint** — a hard **free-first mandate** shaped the architecture and the
   roadmap, not just the budget.
3. **Ship the spine, defer the long tail** — get the end-to-end golden path working, then add depth.

## 5.2 The MVP cut line

Because the prototype was frozen, *every screen ships in the UI* — so "deferred" never meant a missing
screen. It meant the screen's **backend wiring** was stubbed for later while its UI shipped. The cut
line:

| Bucket | Features | Decision |
|--------|----------|----------|
| **MVP — ship the spine** | F1–F13, core reports (F15), search (F16), parties (F17), settings (F19), audit (F22) | Build fully, backend + UI |
| **Deferred to v2** | F14 (live e-invoice), F20 (workforce/attendance), F21 (analytics), long-tail reports | UI shipped, backend stubbed |

This kept MVP focused on the **order → procure → produce → dispatch → invoice → pay → post-to-ledger**
spine — the chain that actually runs the business.

## 5.3 Cost as a prioritization constraint (the free-first mandate)

A defining product constraint: **the app must cost ₹0/month to run, with no payment card attached to
any cloud account** (so it physically cannot incur surprise billing). This wasn't just frugality — for
an MSME, an unexpected cloud bill or a vendor lock-in is a real operational risk.

This constraint actively shaped priorities and architecture:

| Decision | Driven by free-first |
|----------|----------------------|
| **Capacitor over React Native** | Reuses the finished web prototype as-is; builds APKs locally for free, instead of weeks rewriting UI in native primitives |
| **Supabase free tier** | One service for Postgres + Auth + RLS + Storage + Edge Functions; self-hostable later to avoid lock-in |
| **On-device / client-side OCR, PDF, Excel** | No paid OCR or document services |
| **GitHub Actions free minutes** | CI/CD at no cost |
| **iOS explicitly deferred** | The only unavoidable cost (Apple Developer $99/yr) — so iOS is "nice to have", Android ships first |

Every paid item was flagged in the plan; the golden rule — *"do not attach a payment card to any
cloud account"* — was treated as a release requirement.

## 5.4 The agreed v1 build order

After MVP, a v1 closeout audit surfaced 16 remaining features. Rather than build them in discovery
order, I sequenced them by **dependency and value**, and agreed the order explicitly:

```
1. Voucher-template CRUD + Supplier payments   (finance depth, unblocked by the COA fix)
2. Multi-truck inward / dispatch               (share the sequence logic)
3. Machine queue + pickling-loss               (ops depth)
4. Analytics + Workforce                       (insight)
5. Live e-invoice + APK packaging              (left last: GSTN-gated / release gate)
```

A critical sequencing insight: several finance features *looked* done as static screens but were only
**correct after a foundational data-integrity fix** (the chart-of-accounts balance sync — see
[Challenges](06-challenges-and-resolutions.md)). I prioritized that fix first so the features built on
top of it would be trustworthy, not just visible.

## 5.5 The parked-feature decision log

Saying "no / not now" is the hardest part of prioritization. I kept an explicit **parked log** so
deferrals were decisions with reasons, not things that quietly fell through:

| Feature | Status | Why parked |
|---------|--------|------------|
| **Workforce + attendance** (#16) | ⏸️ Parked | Lowest operational urgency for a 2-owner business; the backend hook exists, wiring deferred |
| **Bank Reconciliation (BRS)** (#4) | ⏸️ Parked | Needs a new bank-statement table + reconcile function; revisit after higher-value finance work |
| **Live e-invoice IRN/IRP** (#14) | ⏸️ Parked | **Externally gated** — requires live GSTN/NIC credentials; the app already records IRN/ack as data and can go live behind creds |

Parking #14 is a good example of mature scoping: it's blocked by an **external dependency**, so
building it early would be speculative. The app stores the data shape it needs and is ready to
activate when the credential exists.

## 5.6 Governance: right-sizing process to a 2-user app

I started from an enterprise technical-product-management charter and **deliberately calibrated it
down** to what a 2-user internal MSME app actually needs — keeping the controls that add value and
explicitly marking the rest *deferred*. The kept controls:

- **KPI governance** — the few numbers that matter (delivery, quality gates, correctness).
- **ADR register** — architecture decisions (valuation method, invoice series, money type) recorded so
  they're not silently re-litigated.
- **Risk register (R1–R12)** — written before code, revisited as risks materialized.
- **Cost governance** — the free-first mandate, enforced as a release requirement.
- **Production-readiness gate** — an explicit go-live checklist for the release milestone.

Knowing *what process to skip* for a small internal tool is itself a product-leadership skill —
over-processing a 2-user app would have been waste.

## 5.7 Stakeholder management

**The stakeholders:** two owner-operators — the **Operations owner** (shop floor, Android) and the
**Finance owner** (books, GST). They are also the domain experts and the end users.

**How I managed them:**

| Practice | What it looked like |
|----------|---------------------|
| **Approve-on-parity** | Because the prototype was the agreed contract, stakeholder review was simple and concrete: *"does this screen match what we agreed?"* — yes → approve; drifted → I correct it. This minimized ambiguous feedback loops. |
| **Decisions, not debates** | Open questions were logged (OQ01–OQ14) and brought to stakeholders attached to the work they blocked — so each conversation produced a decision that unblocked specific modules. |
| **Visible status** | A rolling session log + technical changelog meant the owners could always see what shipped, what's next, and what's parked — no black box. |
| **Respecting their reality** | Mobile-first for the floor, finance-on-PC-and-phone, finance data hidden from the ops role, graceful behaviour on poor 4G — the product met them where they actually work. |
| **Confirmation gates** | Key flows (auth, OCR accuracy, the finance trio) were validated *on their device* with their data before being called done. |

**A concrete example of stakeholder-driven change:** the authentication flow's one-time-passcode length
was a stakeholder preference — set to 8 digits, then reverted to **6 at the owners' request**. I traced
that the digit count was a backend setting (not the email template), changed it cleanly, and aligned
the app. Small, but it shows the loop: stakeholder preference → correct root cause → implement →
confirm on their device.

---

[← Previous: Requirements → User Stories](04-requirements-to-user-stories.md) · [Next: Challenges & Resolutions →](06-challenges-and-resolutions.md)
