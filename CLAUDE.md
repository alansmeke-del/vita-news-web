# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Static HTML landing page for **Vita News** (vitanews.com.mx), a Spanish-language business newsletter. The entire site is a single file: `index.html` (~960 lines) with embedded CSS and JavaScript. There is no build system, package manager, or framework.

**Deployment**: Git push to `main` → GitHub Pages auto-publishes to vitanews.com.mx (CNAME file).

## Development

No build step. To preview locally, open `index.html` directly in a browser or use any static file server:

```bash
python3 -m http.server 8080
```

## Architecture

Everything lives in `index.html`, organized in this order:

1. **`<head>`** — Google Fonts (Libre Baskerville, IBM Plex Sans, IBM Plex Mono), embedded `<style>` block
2. **Embedded CSS** — CSS custom properties (`--ink`, `--paper`, `--gold`, `--red-accent`, etc.) drive the color system. Responsive via `clamp()` fluid typography and a `@media (max-width: 430px)` breakpoint for very small screens.
3. **Page sections** (in DOM order): masthead → nav → hero → problem/value → pricing → testimonial → footer → subscription modal overlay
4. **Inline `<script>`** at the bottom handles:
   - Dynamic Spanish date in the masthead
   - Free-tier modal open/close logic
   - Email subscription POST to Railway proxy (`gentle-laughter-production-7f2c.up.railway.app/subscribe`), which forwards to the MailerLite API (group ID `187372665902728776`)
5. **Stripe** checkout links are plain `<a href>` tags pointing to Stripe-hosted payment pages.

## Key Integrations

| Integration | How it's wired |
|---|---|
| Email subscriptions (free tier) | POST to Railway proxy → MailerLite group |
| Premium plan checkout | Static Stripe-hosted URL in anchor tag |
| Domain | `CNAME` file → vitanews.com.mx via GitHub Pages |

## Conventions

- All user-facing copy is in **Spanish**.
- Typography hierarchy uses `clamp()` — avoid fixed `px` font sizes.
- Color changes should use the CSS variables defined at the top of the `<style>` block, not hardcoded values.
