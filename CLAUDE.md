# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

One-page marketing site for "Horizon Wealth Planning", a financial planning firm. Everything lives in `index.html`: HTML, CSS in a single `<style>` tag, and vanilla JavaScript in a single `<script>` tag at the end of `<body>`. This is a hard constraint: no frameworks, build tools, package manager or extra files.

## Running

There is no build, lint or test tooling, and Node is not installed on this machine. To view the site, open the file in the default browser:

```powershell
Start-Process "index.html"
```

The hero image (Unsplash), the fonts (Google Fonts) and the avatars (pravatar.cc) load from the internet, so they don't render offline.

## Architecture

### CSS (`<style>`)
- Design tokens are custom properties on `:root`: colours (`--navy #0B2545`, `--gold #C9A227`, `--bg #F7F9FC`), the spacing scale `--space-1`…`--space-9`, radii, shadows and `--nav-h`. Use these tokens instead of hard-coded values.
- The CSS is mobile-first. The base rules target mobile, and the only breakpoints are `@media (min-width: 768px)` and `@media (min-width: 1024px)`, each in a single block near the end of the stylesheet. Add responsive overrides to those blocks rather than creating new media queries.
- The header is `position: sticky` with height `--nav-h`, and `--nav-h` changes at 768px. `html { scroll-padding-top: var(--nav-h) }` stops anchor jumps from hiding under the header, and the hero uses `min-height: calc(100vh - var(--nav-h))`.
- `.reveal` elements start hidden and fade in when JS adds `.is-visible`. A `<noscript>` style and the `prefers-reduced-motion` block both force them visible, so new `.reveal` content has to keep working in both cases.

### JS (`<script>`, one IIFE in numbered sections)
1. Footer year
2. Mobile nav: `setMenu(open)` keeps `aria-expanded`/`aria-label` on `.nav-toggle` in sync with `.is-open` on `#nav-menu`
3. Scroll effects: header shadow, back-to-top button shown after 400px
4. Active nav link, via IntersectionObserver on `main section[id]`
5. Fade-in reveal
6. Stat counters, driven by `data-target`/`data-prefix`/`data-suffix` on `.stat-number`. The HTML holds the final values as a no-JS fallback, and screen readers read a sibling `.sr-only` copy.
7. Testimonial carousel, described below
8. Hero CTA scroll (`data-scroll-to="<id>"`)
9. Enquiry form validation and submission
10. Newsletter signup

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
