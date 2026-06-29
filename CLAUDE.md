# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Static HTML landing page for **A LAS 7** (alas7.com), a Spanish-language business newsletter. The entire site is a single file: `index.html` (~1020 lines) with embedded CSS and JavaScript. There is no build system, package manager, or framework.

**Deployment**: Git push to `main` → GitHub Pages auto-publishes to alas7.com (CNAME file).

## Development

No build step. To preview locally, open `index.html` directly in a browser or use any static file server:

```bash
python3 -m http.server 8080
```

## Architecture

Everything lives in `index.html`, organized in this order:

1. **`<head>`** — Google Fonts (Libre Baskerville, IBM Plex Sans, IBM Plex Mono), embedded `<style>` block
2. **Embedded CSS** — CSS custom properties (`--ink`, `--paper`, `--gold`, `--red-accent`, etc.) drive the color system. Responsive via `clamp()` fluid typography and two breakpoints: `@media (max-width: 640px)` collapses the hero/plans/preview grids from two columns to one; `@media (max-width: 430px)` fine-tunes spacing and makes the nav a single horizontal scroll row.
3. **Page sections** (in DOM order): masthead → nav → hero → preview → pricing → testimonial quote → footer → subscription modal overlay
4. **Inline `<script>`** at the bottom handles:
   - Dynamic Spanish date in the masthead
   - Referral code capture: reads `?ref=` from the URL on load and stores it in a hidden `#referralCode` input; the value is sent in the MailerLite subscribe payload
   - Free-tier modal open/close logic (closes on overlay click or ✕ button; resets privacy checkbox state on close)
   - Privacy checkbox gate: the free modal submit button starts `disabled` and is only enabled after the checkbox fires `toggleFreeBtn()`; the premium button calls `checkPremiumPrivacy()` before redirecting to Stripe
   - Email subscription POST to Railway proxy (`gentle-laughter-production-7f2c.up.railway.app/subscribe`), which forwards to the MailerLite API (group ID `187372665902728776`)
5. **Stripe** checkout links are plain `<a href>` / `window.location.href` pointing to a Stripe-hosted payment page (`buy.stripe.com/14AcN50xF8OM2OW4SsafS01`).

## Key Integrations

| Integration | How it's wired |
|---|---|
| Email subscriptions (free tier) | POST to Railway proxy → MailerLite group |
| Premium plan checkout | `checkPremiumPrivacy()` → `window.location.href` to Stripe URL |
| Referral tracking | `?ref=` URL param → hidden input → subscribe payload field `referral_code` |
| Domain | `CNAME` file → alas7.com via GitHub Pages |

## Conventions

- All user-facing copy is in **Spanish**.
- Typography hierarchy uses `clamp()` — avoid fixed `px` font sizes.
- Color changes must use the CSS variables defined at the top of the `<style>` block, not hardcoded values.
- The privacy checkbox is a hard gate for both conversion flows. Any new subscription or payment button must go through the same checkbox → error message → action pattern.
