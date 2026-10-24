# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

One-page marketing site for "Horizon Wealth Planning", a financial planning firm aimed at women, with a soft Eastern look (plum blossoms, ink-wash mountains, red seal characters) and a playful tone. Its main job is lead capture: a free guide, a quiz and a popup, all collecting emails. Everything lives in `index.html`: HTML, CSS in a single `<style>` tag, and vanilla JavaScript in a single `<script>` tag at the end of `<body>`. This is a hard constraint: no frameworks, build tools, package manager or extra files.

## Running

There is no build, lint or test tooling. Node.js (LTS) is installed only to run the Playwright MCP server configured in `.mcp.json`; the site itself must not depend on it. To view the site, open the file in the default browser:

```powershell
Start-Process "index.html"
```

Only the fonts (Google Fonts) load from the internet. All artwork is inline SVG/CSS and avatars are initials. The seal font (Noto Serif SC) is subset with `&text=`, so a new Chinese character must also be added to that URL or it falls back to a system font.

Playwright MCP can't open `file:` URLs. To test in it, serve the folder on localhost (e.g. a one-line Node `http` server) and navigate to that. The guide popup is suppressed when `sessionStorage['hwp-guide-popup']` is set, which is useful before full-page screenshots.

## Deployment

`.github/workflows/pages.yml` deploys on every push to `main`. The original site is frozen at git tag `v1` and stays at the root URL (https://tybaltybalt-wq.github.io/financialplanning-trial/). The current `index.html` is published at `/v2/`. Never move or delete the `v1` tag.

## Architecture

### CSS (`<style>`)
- Design tokens are custom properties on `:root`: colours (`--plum #4A2545` for headings and dark panels, `--rose #B83B5E` for CTAs and accents, `--blush`, `--peach`, `--gold`, `--jade`, `--bg #FFF9F5`, plus `--spring`/`--summer`/`--autumn`/`--winter` for the season cards), fonts (`--font-heading` Cormorant Garamond, `--font-body` Nunito, `--font-seal` Noto Serif SC), the spacing scale `--space-1`…`--space-9`, radii, shadows and `--nav-h`. Use these tokens instead of hard-coded values.
- The CSS is mobile-first. The base rules target mobile, and the only breakpoints are `@media (min-width: 768px)` and `@media (min-width: 1024px)`, each in a single block near the end of the stylesheet. Add responsive overrides to those blocks rather than creating new media queries.
- The header is `position: sticky` with height `--nav-h`, and `--nav-h` changes at 768px. `html { scroll-padding-top: var(--nav-h) }` stops anchor jumps from hiding under the header, and the hero uses `min-height: calc(100vh - var(--nav-h))`.
- `.reveal` elements start hidden and fade in when JS adds `.is-visible`. A `<noscript>` style and the `prefers-reduced-motion` block both force them visible, so new `.reveal` content has to keep working in both cases. The same blocks hide the quiz card (no JS) and the falling `.petals` (reduced motion).
- The plum blossom is one SVG `<symbol id="blossom">` at the top of `<body>`, reused via `<use href="#blossom" color="…">` in the logo, hero branch, book covers and contact panel. The book cover (`.guide-cover`, `--sm` variant in the popup) is pure CSS and appears three times; keep the copies in step.
- Buttons: `.btn-rose` (primary), `.btn-ghost` (outline), `.btn-plum`.

### Page sections (in order)
Hero (`#home`) → Free guide (`#guide`) → Seasons of life services (`#services`) → Money Flower quiz (`#quiz`) → Testimonials (`#testimonials`) → Tea chat enquiry form (`#contact`), then the footer and the `<dialog id="guide-dialog">` popup.

### JS (`<script>`, one IIFE in numbered sections)
1. Footer year
2. Mobile nav: `setMenu(open)` keeps `aria-expanded`/`aria-label` on `.nav-toggle` in sync with `.is-open` on `#nav-menu`
3. Scroll effects: header shadow, back-to-top button shown after 400px
4. Active nav link, via IntersectionObserver on `main section[id]`
5. Fade-in reveal
6. Stat counters, driven by `data-target`/`data-prefix`/`data-suffix` on `.stat-number`. The HTML holds the final values as a no-JS fallback, and screen readers read a sibling `.sr-only` copy.
7. Testimonial carousel, described below
8. CTA scroll (`data-scroll-to="<id>"`). An optional `data-interest="<option value>"` also pre-selects that topic in the enquiry form, as the season cards do.
9. Enquiry form validation and submission
10. Newsletter signup
11. Lead capture forms: `setupCaptureForm(form, { source, container, title(name), text })` wires the guide, quiz and popup forms. It reads an optional `firstName`, a required `email` and an optional hidden `flower`. It logs the lead as JSON, marks the popup as `claimed`, and swaps `container` for a success message built with `textContent`.
12. Money Flower quiz: one `.quiz-step` per question; each `.quiz-option` has `data-flower` (`plum`/`peony`/`lotus`/`orchid`). The flower picked most often wins, and ties go to the first answer. The result copy lives in the `FLOWERS` object.
13. Guide popup: opens once per session on exit intent (pointer leaves the top of the window) or at 60% scroll depth, but never in the first 8s, while the mobile menu is open, or after any lead form succeeded.

**Carousel:**
- Cards per view are 1, 2 or 3. `getPerView()` reads them from `matchMedia` queries that mirror the CSS `flex-basis` of `.carousel-slide` at the same breakpoints, so change both together.
- Positions run `0..slides.length - perView` and wrap at the ends. Dots are rebuilt by `buildDots()` whenever the breakpoint changes.
- Autoplay runs every 5s and pauses on hover, while focus is inside the carousel, during a touch, and while the tab is hidden.
- The star ratings are injected by JS into the empty `.stars` containers.
- To add a testimonial, copy an existing `.carousel-slide` block. The slide labels and dots are generated automatically.

**Enquiry form:**
- Each field's wrapper has `data-field="<name>"`, and its error element has the id `<name>-error`.
- `validators[name]()` returns an error message, or `''` when the field is valid. `setFieldState` toggles `.has-error` on the wrapper and `aria-invalid` on the controls.
- To add a field, add the wrapper and the error `<p>`, then add a validator entry. Live validation and submit handling pick it up automatically.
- Phone numbers must be digits only, 8–15 digits.
- A valid submit shows a spinner for 1.5s, logs the data with `console.log` as JSON, then replaces the contents of `#form-card` with a success message. The user's name is inserted with `textContent`, never as HTML.
