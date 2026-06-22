# 9 · Showcase Guide — How This Repo & Website Were Set Up

[← Back to index](README.md)

---

This repository **is** the showcase: a clean, public, anonymized extract of the case-study pack, served
as a website by **GitHub Pages** directly from the Markdown. This page documents how it's wired so it
can be reproduced or maintained.

> This is the *public* repo. The application's private build repository (source, migrations, CI, secrets)
> is kept separate and private — only this curated case-study pack is published.

---

## Part A — The repository

- **Layout is root-served:** the executive-summary `README.md`, the nine numbered docs, the
  `08-product-docs/` artefacts (PRD/BRD/User-Stories/Case-Study), and `screenshots/` all live at the
  repo root. Every internal link is relative, so the same files render correctly both on the GitHub
  repo page and on the published site.
- **`README.md` = executive summary + landing page.** It opens with the hook, a live-site link, badges,
  a "by the numbers" table, a screenshots gallery, and the document index. It doubles as the Pages
  homepage.
- **Discoverability:** on the repo's main page, click the ⚙️ next to **About** and set a description, the
  **website** (the Pages URL), and **topics** — e.g. `erp`, `react`, `typescript`, `supabase`,
  `capacitor`, `postgres`, `mobile-app`, `gst`, `double-entry-accounting`, `product-management`,
  `case-study`. Then **pin** the repo to your GitHub profile.

## Part B — The website (GitHub Pages), click-by-click

Pages publishes this repo at `https://<username>.github.io/steel-erp-case-study/` — free, no extra
account. This repo serves Pages **from the repository root** (not a `/docs` folder).

1. Push this repo to GitHub as a **public** repository named `steel-erp-case-study`.
2. On GitHub, open **Settings** → (left sidebar) **Pages**.
3. Under **Build and deployment → Source**, choose **Deploy from a branch**.
4. Under **Branch**, select **`main`** and the folder **`/ (root)`**, then click **Save**.
5. Wait ~1–2 minutes, then refresh — you'll see **"Your site is live at
   `https://<username>.github.io/steel-erp-case-study/`"**. Open it; the `README.md` renders as the
   homepage with the theme.
6. Paste that URL back into the repo's **About → Website** field.

### Theme

`_config.yml` at the repo root sets the title, description, author, and a built-in theme
(`jekyll-theme-cayman` by default; try `minimal`, `architect`, `slate`, or `merlot` by changing the one
`theme:` line, committing, and waiting a minute).

### `.md` links on Pages

GitHub Pages enables the `jekyll-relative-links` plugin by default, so relative links to `.md` files
(e.g. `01-product-overview.md`, `08-product-docs/README.md`) are rewritten to their generated `.html`
pages automatically — the same links work in the repo view and on the live site.

### Custom domain (optional)

Settings → Pages → **Custom domain** → enter your domain → add the DNS records GitHub shows (a `CNAME`
to `<username>.github.io`) at your registrar → tick **Enforce HTTPS** once it validates.

## Part C — A note on data

- The written prose anonymizes business identifiers (owner names, GSTIN, exact address).
- The **screenshots show authentic operational data**, which the business owners explicitly approved for
  a partial public showcase. See [`screenshots/README.md`](screenshots/README.md).

---

## Quick checklist

- [x] Root README = executive summary (hook + live-site link + badges + gallery + doc index)
- [x] `_config.yml` (title/description/author/theme) at root
- [x] `screenshots/` populated + captioned
- [ ] Repo created on GitHub as **public** `steel-erp-case-study`, pushed
- [ ] About → description, topics, website set; repo pinned to profile
- [ ] Settings → Pages enabled (branch `main`, `/ (root)`)
- [ ] Live URL verified and pasted into About → Website

---

[← Back to index](README.md)
