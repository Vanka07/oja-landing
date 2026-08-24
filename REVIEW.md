# Oja landing review (for Jay)

Reviewed `main` at `bb9ac00` plus the live marketing hosts on 2026-08-24. This is a read of the repo and the served pages, not a rewrite. Nothing was deployed.

**Verdict:** `oja-landing` is a static Vercel marketing site. Product signup, plans, and any real billing live (or fail to live) in a separate Expo web app at `https://app.ojapos.app`. This repo does not collect money.

---

## Method

- Read every tracked file in this repo (no `package.json`, no `vercel.json`, no README).
- Grepped for pricing copy, CTAs, Paystack/billing, `/docs`, and outbound URLs.
- Fetched live `ojapos.app` / `ojapos.ng` headers and pages. Both are `server: Vercel`, same homepage `etag` (`bd10a462…`) and `content-length` (59870 vs local `index.html` 59871).
- Confirmed `/docs` is Vercel `NOT_FOUND` on both hosts.
- Did **not** reverse the Expo bundle on `app.ojapos.app` beyond confirming it is a separate JS app.

---

## Page map

| URL (live clean path) | File | In nav/footer? | In `sitemap.xml`? | Role |
| --- | --- | --- | --- | --- |
| `/` | `index.html` | yes | yes | Home / marketing |
| `/features` | `features.html` | yes | yes (`/features.html`) | Feature brochure |
| `/pricing` | `pricing.html` | yes | yes (`/pricing.html`) | Plans + comparison table |
| `/faq` | `faq.html` | yes | yes (`/faq.html`) | FAQ + FAQPage JSON-LD |
| `/about` | `about.html` | yes | yes (`/about.html`) | Story / values |
| `/privacy` | `privacy.html` | footer only | yes (`/privacy.html`) | Privacy policy (Feb 2026) |
| `/catalog` | `catalog.html` | **no** | **no** | Hash-payload WhatsApp catalog viewer |
| `/payment-callback` | `payment-callback.html` | **no** | **no** | Paystack return **UI stub** |
| `/font-preview` | `font-preview.html` | **no** | **no** | Internal font bake-off, public |
| `/docs` | *none* | no | no | **404** (`x-vercel-error: NOT_FOUND`) |
| `google04f7fd945c7da525.html` | same | no | no | Search Console verify |

Vercel clean-URL rewrite: `/pricing.html` 308 → `/pricing` (same for other `.html` files). Nav still links the `.html` forms; they work.

`robots.txt` allows `/` and points at `https://ojapos.app/sitemap.xml`. Sitemap `lastmod` is 2026-02-01; it also lists `https://app.ojapos.app`.

`font-preview.html` is leftover design work. It is live.

---

## Pricing copy (what the site claims)

Canonical three-tier copy on `/` and `/pricing`:

| Plan | Price | Claimed extras |
| --- | --- | --- |
| Starter | Free forever | Unlimited sales, 50 products, basic reports (today), 1 staff, WhatsApp receipts + storefront, offline, credit book. Pricing page also lists barcode + cash/transfer/POS. |
| Growth | ₦2,500 / month | Unlimited products, advanced reports/charts, **cloud backup & sync**, weekly/monthly/yearly views (pricing page). |
| Business | ₦5,000 / month | Unlimited staff, payroll, low-stock WhatsApp alerts, receipt printer, priority support. |

JSON-LD on `index.html` and `pricing.html` repeats those prices (`NGN` 0 / 2500 / 5000).

Growth is labeled **Popular**. Business is labeled **Best Value** and still has leftover CSS `.pricing-card.featured::before { content: 'Most Popular'; }` on both home and pricing, so the Business card can show two badges stacked at the same `top: -14px`.

---

## CTA map — all product CTAs go to the app, none to checkout

Every “Try Free / Get Started / Start Free Trial” button is `https://app.ojapos.app` with **no** query (`?plan=`, `?checkout=`, etc.).

| Surface | Label | Target |
| --- | --- | --- |
| Nav (all marketing pages) | Try Free | `https://app.ojapos.app` |
| Home hero | Try Oja Free | same |
| Home hero / footer CTA | Share on WhatsApp | `https://wa.me/?text=…` (share-to-anyone, **no** phone number; body promotes `app.ojapos.app`) |
| Home / pricing cards | Get Started Free / Start Free Trial ×3 | `https://app.ojapos.app` (Growth and Business use the same URL as Starter) |
| Features / FAQ / About / pricing bottom | Try / Get Started | same |
| Pricing “Need help” | Chat on WhatsApp | `https://wa.me/2348022471137?text=…pricing` |
| Pricing / About / FAQ | Email | `mailto:hello@ojapos.app` |
| About secondary | Chat With Us | `wa.me/2348022471137` |
| Footer (all pages) | email, WhatsApp +234 802 247 1137, X `@ojaposapp`, IG `@ojaposapp` | outbound |

There is **no** Stripe/Paystack checkout form, no plan id, no “Buy Growth” deep link.

Live homepage CTA count matches source: **6** `href="https://app.ojapos.app"`.

---

## Billing / payment code in this repo

**Marketing only**

- Home trust tile: “Secure payments via Paystack” (`index.html` ~1265). Not a button. Ambiguous (merchant Paystack vs Oja subscription).
- FAQ upgrade answer: “upgrade directly within the app” **or** WhatsApp; “bank transfer and other payment methods”; “free trial of the Business plan.” No in-page pay flow.
- Privacy: optional cloud sync “powered by Supabase.” No billing.

**The only payment-shaped page:** `payment-callback.html`

- Reads `?reference=` or `?trxref=` (Paystack redirect params).
- If either is present → **always** “Payment Successful!” and “Your Oja POS **Business** plan has been activated.”
- If missing → “Payment Failed.”
- Comment in source: *“If we have a reference, the payment likely went through (Paystack redirects on success).”*
- Does **not** call Paystack, Supabase, or any API. Does not activate a plan. Does not mention Growth.
- Not linked from nav, footer, or sitemap. Live `/payment-callback` (no query) renders the failed state, as the script intends.

**Catalog helper (not billing):** `catalog.html` decodes `#data=` (base64 JSON: shop, desc, WhatsApp number, products) and builds `wa.me` order links. Empty hash → “Catalog Not Found” + CTA to `ojapos.app`. No server. This is a storefront renderer the **app** would populate; it is not a checkout for Oja itself.

**Not present:** Paystack public key, secret, webhook, Stripe, Flutterwave, prices API, entitlements, auth, env vars (`.gitignore` has `.vercel` and `.env*.local` only).

`app.ojapos.app` is a **different** Vercel deploy: Expo / React Native Web shell (`#_expo/static/js/web/…`). That is where any real upgrade UI would have to live. This review did not audit that bundle.

---

## Marketing-only vs wired

| Claim / surface | In this repo | Wired to product/billing? |
| --- | --- | --- |
| Three plans + ₦0 / ₦2,500 / ₦5,000 | yes, copy + JSON-LD | **Copy only.** CTAs ignore plan. |
| “Start Free Trial” on paid cards | yes | Same URL as free. No trial clock here. |
| “No credit card required” | pricing / features | True for this site (there is no card field). |
| Cloud backup on **Growth** | pricing cards + table + cost FAQ | **Copy.** Contradicted elsewhere (below). |
| Cloud backup on **Business** | FAQ body, privacy | **Copy.** Conflicts with pricing. |
| Upgrade in-app / WhatsApp / bank transfer | FAQ | **Not implemented here.** WhatsApp is a real `wa.me` link. |
| Paystack | trust copy + callback stub | **Not a checkout.** Callback is optimistic UI. |
| WhatsApp storefront | marketed as free on Starter | `catalog.html` is a client renderer only. |
| Credit Intelligence | home, features, FAQ | **Not on any plan table.** No price, no CTA. |
| Payroll, printer, low-stock alerts | priced as Business; also ungated on home/features | Marketing. |
| Expense tracking, cash register, price calculator, PIN recovery, Yorùbá/Pidgin/Igbo/Hausa | home only | Ungated marketing. Not on pricing. |
| Advanced reports | Growth on pricing; **Business** in FAQ reports answer | Conflict. |
| Staff: 1 on Starter/Growth, unlimited on Business | pricing table | FAQ skips Growth (free vs Business only). |
| JSON-LD `aggregateRating` 4.8 / 12 | `index.html` | No reviews UI. Unverified here. |
| `/docs` | none | 404. No docs link in this repo either. |
| GA4 `G-9P5LC72768` | most pages | Tracking only. |
| Product app | outbound only | Separate Expo app. |

---

## Copy conflicts (confirmed in source; matches 2026-08-23 live notes)

**1. Cloud backup: Growth vs Business (the live FAQ vs pricing bug)**

| Place | Cloud backup / multi-device sync belongs to |
| --- | --- |
| Home Growth card | Growth |
| Pricing Growth card + comparison row | Growth (Starter ✗, Growth ✓, Business ✓) |
| FAQ “How much does it cost?” (visible + JSON-LD) | Growth |
| FAQ “Is my data safe?” **visible** body | **Business** |
| FAQ “Can I use multiple devices?” visible + JSON-LD | **Business** |
| FAQ “Do I need internet?” visible | **Business** |
| `privacy.html` §3 Optional Cloud Sync | **Business** |
| Pricing JSON-LD Business `description` | still lists “cloud backup” on Business (and “advanced reports”), as if Growth does not exist |

JSON-LD for “Is my data safe?” does **not** name a plan; the visible paragraph does.

**2. Advanced reports:** pricing/Growth vs FAQ “What reports can I get?” (Business).

**3. Credit Intelligence:** sold as a product capability, missing from the plan matrix. FAQ says “Upgrade to Credit Intelligence” with no price or plan.

**4. Features page** lists cloud backup, payroll, printer, staff, smart reports **without plan tags**, so the brochure reads as “included.”

Not fixed in this PR: which plan is the source of truth is an app/entitlement question, not a one-line landing typo.

---

## Other notes (not blocking the review)

- Repo is 5 commits (2026-02-02/03): initial pages, GA, two theme experiments, revert to `#0c0a09`. No prior PRs/issues.
- Home/pricing/FAQ/about/privacy share a broken CSS nest: heading `font-family` rules are inside `body { }` and the `line-height`/`overflow` declarations sit after a nested block. Browsers still paint; it is leftover paste, not a second site.
- Sitemap omits `/catalog` and `/payment-callback` (reasonable if they are app helpers) and `font-preview` (should not be public).
- Canonicals and OG URLs are `ojapos.app` only. `ojapos.ng` is the same files with no `.ng` canonical. Fine if `.ng` is an alias; search may prefer `.app`.
- `payment-callback` success copy hard-codes **Business**, so a Growth Paystack return (if the app ever used this page) would lie.

---

## What this repo is not

It is not the POS, not a docs site, and not a billing service. Wiring “Start Free Trial” to a real Growth/Business checkout, syncing FAQ/privacy with entitlements, and verifying Paystack would be work in **`app.ojapos.app`** (and whoever owns Paystack keys), not a landing rewrite.

---

## Live facts (2026-08-23 brief + 2026-08-24 recheck)

| Fact | Recheck |
| --- | --- |
| `ojapos.app` and `ojapos.ng` same Vercel marketing page | Yes. Same etag, same body. |
| Starter Free / Growth ₦2,500 / Business ₦5,000 | Yes, home + pricing + schema. |
| All CTAs → `https://app.ojapos.app`, no checkout | Yes. |
| FAQ says cloud backup is Business; pricing says Growth | Yes; also privacy + FAQ JSON-LD multi-device. |
| `/docs` 404 | Yes on both hosts. No `docs` file in repo. |
