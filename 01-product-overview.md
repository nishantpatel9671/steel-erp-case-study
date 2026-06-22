# 1 · Product Overview & Target Users

[← Back to index](README.md)

---

## 1.1 The product in one line

**Steel ERP** is an internal, mobile-first ERP that runs the day-to-day operations and finance of a
**steel round-bar processing MSME** in Gujarat, India — replacing paper registers, spreadsheets, and
WhatsApp coordination with a single, auditable system that calculates money, GST, and inventory
exactly.

## 1.2 The problem it solves

The business is a small/medium manufacturer that buys raw steel bars, processes them on **draw
benches** into finished "bright bars", and sells them — while also doing **job-work** (processing a
customer's own material for a fee). Before the app, the entire operation ran on:

- Paper order books, handwritten job cards, and physical bill files.
- Spreadsheets for stock and pricing that drifted out of sync.
- WhatsApp messages for coordinating trucks, dispatches, and payments.

This caused the usual MSME pain: **no single source of truth, error-prone money/GST maths, no audit
trail, and no real-time view** of stock, receivables, or production. Mistakes in GST treatment
(especially the difference between a sale and job-work) carry real compliance and cash risk.

**Steel ERP digitizes the whole chain** — from a customer order to the cash received against it — with
the correct business rules enforced by the database, not by memory.

## 1.3 Two business models, one system

The single hardest product insight is that this company runs **two fundamentally different business
models** that must coexist without contaminating each other's books:

| Model | Flow | Billing | Stock & GST impact |
|-------|------|---------|--------------------|
| **Purchase–Sale** | Buy raw bars → draw/process → sell finished bars | Full **GST invoice** on goods | Material enters the company's stock value; GST input credit applies |
| **Job-Work** | Customer's material comes in → process → return | **Charges-only** invoice (SAC, conversion fee) | Customer material **never** enters the company's stock value; **no** GST input credit on it |

If these two are ever confused — e.g. claiming GST input on job-work material, or valuing a customer's
steel as company stock — the result is wrong invoices, wrong inventory valuation, and a real legal/tax
exposure. The product treats `order_type` as **sacred** and enforces the separation at the database
level.

## 1.4 Six fulfillment scenarios

A single sales order line can be fulfilled in one of six ways, decided per line as a `srcType`:

1. **stock** — ship straight from finished-goods stock (no production).
2. **produce** — make it: consume internal raw material on a draw bench via a Job Card.
3. **procure** — buy raw material first (Supplier PO → Inward), then produce.
4. **trade** — buy finished goods and resell them (no production).
5. **jobwork** — process the customer's own material; bill conversion charges only.
6. **partial** — a single line split across the above (e.g. part from stock, part produced).

The app models this with a **fulfillment plan** per order and surfaces a "flow scenario" callout so the
operator always knows which path a line is on.

## 1.5 What the app covers (modules)

| Area | Capabilities |
|------|--------------|
| **Orders** | Sales orders, fulfillment planning, per-line srcType, full document chain (PO → Supplier PO → Inward → Job Card → Dispatch → Payment) |
| **Procurement & Inward** | Supplier POs, **multi-truck** inward receipts against one PO, multi-item bills, OCR-assisted bill capture |
| **Inventory** | Cost-lot stock ledger, WAC/FIFO valuation, manual adjustments, scrap batches, pickling-loss logging, long-term-storage flags |
| **Production** | Job cards with material/output logs, per-draw-bench queue, die auto-match by diameter, yield/scrap tracking |
| **Dies & Stores** | Dies by internal diameter (with rework that enlarges the bore to the next standard size), consumables, spare bills |
| **Dispatch & GST** | Sale + job-work dispatch, **multi-truck** dispatch, gap-free invoice series, intra/inter-state GST split, E-Way bill prompt above ₹50,000 |
| **Finance** | Chart of accounts, double-entry journal, voucher templates, ledgers, payments received/made, trial balance, P&L, balance sheet, aging |
| **Reporting & Analytics** | Report library with CSV/PDF export, 6-month trend charts, production-output analytics |
| **Cross-cutting** | Universal search, parties (buyers/suppliers) master, notifications/alerts, settings, light/dark theme, immutable audit log |

## 1.6 Target users & personas

The app is **internal and role-based** — not a public SaaS. There are two real daily users (the
owners) plus an admin capability, with restricted roles reserved for future staff.

> Names anonymized; personas describe the real roles.

### 👤 Operations Owner — *"I run the floor."*
- **Device:** Android phone, on the shop floor.
- **Scope:** inwards, job cards, draw benches, dies & stores, dispatch, scrap.
- **Needs:** fast data entry on mobile, clear queues, alerts for low stock / due dies, no finance
  clutter.
- **Role:** `ops` — explicitly **denied** access to finance tables.

### 👤 Finance Owner — *"I keep the books and the GST clean."*
- **Device:** PC + phone.
- **Scope:** GST, bills, job-work charges, payments, banking, vouchers, analytics.
- **Needs:** exact money and tax, balanced ledgers, receivable/payable aging, statements.
- **Role:** `finance` — full access to finance modules.

### 👤 Admin (either owner) — *"I manage the system."*
- **Scope:** user management, master data, settings, audit log.
- **Role:** `admin` — superset of all access.

### 👤 Worker / Accountant *(future, out of scope today)*
- Restricted, view-mostly roles reserved for when staff are added — the RBAC model already
  accommodates them.

## 1.7 Platform & distribution

- **Android-first** internal app, packaged with Capacitor as a **signed APK** and sideloaded — **no
  Google Play / App Store** distribution.
- **Online-first**, designed to degrade gracefully on poor 4G (caching, retry, clear errors).
- **Single company, single GSTIN** — no multi-tenant/multi-branch complexity.
- **Zero run-cost** — built entirely on free tiers with no payment card attached, eliminating any
  surprise-billing risk.

## 1.8 Why it matters

For an MSME, an off-the-shelf ERP is expensive, over-featured, and rarely fits the specific
draw-bench + job-work + multi-truck reality of this business. Steel ERP is a **purpose-built, exact,
auditable system** that fits the operation precisely, costs nothing to run, and is owned outright by
the business.

---

[← Back to index](README.md) · [Next: My Role & Responsibilities →](02-role-and-responsibilities.md)
