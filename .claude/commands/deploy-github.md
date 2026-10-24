---
description: Commit and push to GitHub, then screenshot the live GitHub Pages site and add it to the README
---

Deploy this site to GitHub Pages and record a screenshot of the live result.

Live URL (current `index.html`): https://tybaltybalt-wq.github.io/financialplanning-trial/v2/

The root URL keeps serving the original site from git tag `v1` and never changes; don't screenshot it or touch `screenshots/live-site.png`.

1. **Review changes.** Run `git status` and `git diff`. Stage only project files (`index.html`, `README.md`, `CLAUDE.md`, `.github/`, `.claude/`, `.mcp.json`, `screenshots/`). Never stage `.docx` files, Word lock files (`~$*`) or `.playwright-mcp/`. If nothing has changed, skip to step 3.
2. **Commit and push.** Commit with a short message describing the change, then `git push origin main`. The push triggers `.github/workflows/pages.yml`, which publishes `index.html` at `/v2/`.
3. **Wait for the deploy.** Wait about 60 seconds, then load the live URL with the Playwright MCP `browser_navigate` tool. If the page doesn't show the latest change yet, wait another 30 seconds and reload; give up after 5 minutes and report it.
4. **Capture the screenshot with Playwright MCP.**
   - `browser_resize` to 1440 × 900, so the carousel shows three testimonial cards.
   - `browser_evaluate`: set `sessionStorage['hwp-guide-popup'] = 'shown'` so the lead popup stays closed, add `is-visible` to every `.reveal` element (otherwise sections below the fold are blank in a full-page capture), set each `.stat-number` to its final value, and wait for `document.fonts.ready`.
   - `browser_take_screenshot` with `fullPage: true` and `filename: "screenshots/v2-site.png"`, replacing the previous one.
   - Read the image and confirm it shows the hero, the free guide, the quiz, three testimonial cards and the enquiry form. Check `browser_console_messages` for errors and report any.
   - `browser_close`.
5. **Update the README.** Make sure `README.md` embeds `screenshots/v2-site.png` under the v2 heading and links the v2 URL.
6. **Push the screenshot.** Commit `screenshots/v2-site.png` and `README.md` with a message ending in `[skip ci]` (the screenshot doesn't change the site, so no redeploy is needed), then push.
7. **Report** the live URL, the commit hashes and the screenshot path.
