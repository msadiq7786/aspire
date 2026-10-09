# rules.md — Aspire Studios → WordPress: Rules & Working Protocol

> **START HERE.** This is the entry point for any AI model picking up this project.
> **Read order:** `rules.md` (this) → `architecture.md` → `design.md` → `tracker.md`.
> These four files replace a full scan of the source. Do **not** re-read `index.html` or the master prompt unless a detail you need is genuinely missing; the line-number map in `architecture.md` §2 tells you exactly where to look.

---

## 0. AI onboarding protocol (follow every session)

1. Read the four files in the order above.
2. Open `tracker.md` → find the **current phase** and **"Next action"** → continue from there.
3. Work **one phase at a time**. When a phase is complete, **stop and wait for the user to say "continue"**.
4. At the end of every phase / session, update `tracker.md`: tick items, log deviations, add handoff notes. Never leave the tracker stale: the next model depends on it.
5. If a requirement here conflicts with `index.html`, the **HTML wins** for look/copy/timings, except items marked **(fix)**.
6. If something can't be verified (e.g. missing assets, can't run WordPress), **say so explicitly** and log it in `tracker.md`. Never claim a test passed that wasn't run.
7. Every phase output must include: files created, deviations from the original, and anything unverified.

---

## 1. Project goal

Convert `index.html` ("Aspire Studios" one-page dark/light agency site) into a production WordPress site that a **non-technical person can maintain**, with **identical design, animations and behaviour**. Seed demo content **exactly** as in the file.

---

## 2. Hard constraints (non-negotiable)

1. **Elementor FREE only.** Never use or require a Pro feature (§6). If something needs Pro, build it in the plugin.
2. **No paid plugins.** Allowed: Hello Elementor, Elementor, Contact Form 7, Flamingo, Safe SVG (or equivalent), WP Mail SMTP or FluentSMTP. No ACF Pro / Pods Pro etc.
3. **Everything editable from wp-admin:** every text, image, link, colour, font, icon and repeated item. No hard-coded copy in JS or PHP templates (defaults are seeded as editable content).
4. **Every section is independent.** Deleting or hiding any section (or item inside it) must never break another section, the layout, the spacing or the JS.
5. **Non-technical maintainability first:** plain-English labels, helper text under every field, no shortcode editing for day-to-day changes.
6. **Identical visuals** in dark and light themes at 1440, 1100, 1000, 900, 800, 600, 560 and 390 px.
7. **Zero JS console errors** in every state, including inside the Elementor editor.

---

## 3. Coding standards

- PHP ≥ 8.0, WP ≥ 6.5, WordPress Coding Standards, text-domain `aspire-core`.
- Prefix **everything** with `as_` (PHP) / `as-` (CSS/classes). **No global element selectors** (`nav`, `section`, `h1`, `a`, `button`, `*{margin:0}`…) and no short generic classes (`.wrap .f .ok .err .in .on .hide .tag .row`) and no short IDs (`#n #s #p #m #bar #nav #ring`).
- Scope all CSS under `.as-sec` / `.as-nav` / `.as-` classes. Keep high-specificity font rules off Elementor widgets so per-widget font choices still win.
- Anchor IDs are the **only** IDs allowed (see `architecture.md` §4.1). JS hooks use `data-as="…"`; brand hover uses `data-network="…"`.
- Nonces + capability checks on every save; sanitise on input; escape on output.
- Use `color-mix(in srgb, var(--accent) N%, transparent)` instead of hard-coded accent RGBA. SVG fills use `currentColor` / CSS vars.
- JS: no jQuery; module registry; every module in `try/catch`, null-guarded, queries only inside its own root.
- Stagger delays via classes `as-d1…as-d6`, **never** inline `style="--d:…"` (WP strips it for some roles; Elementor Free can't set it).
- Don't use inline custom properties (`style="--x:…"`) or `<svg>/<iframe>/<style>/<script>` in editable content (KSES strips them). Render those only from plugin PHP.

---

## 4. Section independence rules

1. One top-level Elementor container per section, fixed CSS ID + classes (`architecture.md` §4.1).
2. No cross-section markup/CSS/ID/JS dependency. No sibling selectors across sections. No negative-margin overlaps. Pseudo-elements stay inside (`overflow: clip`).
3. **Spacing is not coupled:** each section gets symmetric `padding-block` = half the original gap per side; one variable `--as-gap` (default 120px).
4. **JS = module registry** (`AS.register`); a missing element must never stop other modules.
5. Nav + footer links **auto-hide** when their target section is absent. No dead anchors.
6. Collections degrade gracefully at **0, 1, 2 and N** items. Empty → render nothing.
7. Shared design-system classes (`.as-pill .as-btn .as-frame .as-rv`) stay global and section-independent.

---

## 5. The 30 pitfalls — required fixes (condensed)

| # | Problem | Required fix |
|---|---|---|
| 1 | Monolithic script; missing element kills all later code | Module registry + per-module try/catch + root guards; scroll-spy built from real links |
| 2 | Arrays sized to today's content (hero 6, services 4, team 3 poses; `art[]`; `serviceImages[]`; `i===1` secondary image) | Pose tables reuse last pose hidden; per-item image fields; no index special-cases |
| 3 | Content lives in JS (marquee, team, testimonials, service images, cursor names) | CPTs rendered **server-side in PHP**; JS only enhances |
| 4 | Global selectors collide with Elementor/WP | Prefix all; scope under `.as-`; no element selectors / universal reset |
| 5 | Elementor defaults (container padding 10, gap 20, widget margin 20, kit typography) | Reset in Site Settings + plugin CSS at `.as-sec` specificity; clamp sizes in plugin CSS |
| 6 | No Form widget in Free; CF7 wraps controls, injects `<p>/<br>`, shows own tips, never clears errors, no storage | CF7 + wrap-aware CSS, autop off, `placeholder " "`, custom `<small>` errors, JS error-clear, Flamingo, honeypot, SMTP notice |
| 7 | No Theme Builder in Free | Plugin renders header/footer/FAB |
| 8 | No Custom CSS / Fonts / Motion Effects / Dynamic Tags / Loop Grid / Popup in Free | Plugin CSS/JS, CPTs + shortcodes, Settings → Advanced CSS, Google Fonts via Settings |
| 9 | `.as-rv` starts `opacity:0` → blank in editor / no-JS / delayed JS | Visible in editor + no-JS; 3s failsafe; re-init on `element_ready`; never combine with Elementor Entrance Animation |
| 10 | Theme applied at end of `<body>` → flash | Inline `<head>` script, priority 1 |
| 11 | FA JS build (`autoReplaceSvg:"nest"`); Elementor's FA lacks `fa-x-twitter` | Self-host FA 6.5.2 **CSS + webfonts**; never load JS build |
| 12 | `logo.svg` blocked by WP uploads | Safe SVG plugin or PNG/WebP fallback; Site Identity logo with bundled default |
| 13 | `sadiq.ico` used as portrait | Convert to JPG/WebP |
| 14 | Relative `assets/…` paths and `--img:url()` inline styles | Media Library `<img>`/featured images; plugin-relative URLs; importer copies demo images into Media Library |
| 15 | Admin bar overlaps fixed nav + progress bar | `body.admin-bar` offsets (32px desktop / 46px mobile) |
| 16 | `fixed`/`sticky` break under transform/filter/overflow ancestors | Print nav/FAB directly under `<body>`; `overflow: clip` in FAQ ancestors; keep Elementor containers `overflow: visible` there |
| 17 | Inter-section padding coupled (Why 0 top, Stats 0, Founder 140/120) | Symmetric half-gap padding (§4.3) |
| 18 | Hero headline reveal needs per-line wrapper spans | Admin types normal heading with line breaks; JS wraps lines at runtime; add `as-split-ready` before animating |
| 19 | Marquee depends on 4× repeat + fixed 38s | Clone-to-fit + width-based duration (constant px/sec) |
| 20 | Testimonials assume 3 visible and ≥4 items; hard-coded stars/Google icon; initials crash on double spaces; no resize handling | §6.8 fixes in `design.md`/`architecture.md`; ignore empty words; guard 0/1/2 items |
| 21 | Cache/optimisation plugins delay JS, combine CSS, inline images | Exclude `aspire.min.js` + inline theme script; noise SVG as a file; Settings "Compatibility" notice; 3s failsafe |
| 22 | KSES strips `<svg>`, `<iframe>`, `<style>`, `<script>`, `style="--x"` for non-`unfiltered_html` users | Render those only from plugin code; use classes |
| 23 | Brand hover colours via `a[aria-label=YouTube]` | `data-network` attributes |
| 24 | Hard-coded accent RGBA in ~10 places and SVG fills | `color-mix()` tokens; `currentColor` |
| 25 | Naming inconsistent ("Aspire Studios" vs "Aspire Digital Solutions"); social links all `#`; "View all services" self-links | Business name + alternate name in Settings; empty URL = hidden; every URL editable |
| 26 | Dead code in source | **Do not port**, and build no admin options for it (`design.md` §10) |
| 27 | Team autoplay stops forever after first hover; infinite rAF/setInterval run off-screen | Pause/resume via IntersectionObserver + `visibilitychange`; resume on leave |
| 28 | Hero uses `100vh` | `100svh` with `100vh` fallback |
| 29 | Source breakpoints (900/1000/1100/800/600/560) ≠ Elementor's (1024/767) | Layout/responsiveness stays in plugin CSS; Elementor only for content, colour, images |
| 30 | Elementor Free container-link / sticky / absolute positioning unreliable | Stat cards, stacks, overlays are plugin-rendered |

---

## 6. Elementor Free vs Pro

**Pro-only — never use:** Form widget · Theme Builder · Popup Builder · Posts / Loop Grid / Loop Carousel · Dynamic Tags + ACF integration · Custom CSS (per-element and Site Settings) · Custom Fonts · Motion Effects · Sticky · Nav Menu / Mega Menu · Slides / Testimonial Carousel / Price Table / Flip Box / Call To Action / Animated Headline / Countdown / Lottie · Global widgets · Role Manager · Custom Code.
(Verify in the installed version; if a control is locked, use the plugin approach.)

**Free widgets to use:** Container, Heading, Text Editor, Image, Button, Icon, Icon Box, Icon List, Social Icons, Divider, Spacer, Shortcode, HTML (static markup only), Menu Anchor.

---

## 7. Accessibility, performance, SEO rules

- Keep: `lang`, `aria-label` on icon-only buttons and social links, `aria-expanded` on burger/FAB, `aria-hidden` on decorative items, `tabindex=0` on service rows, `rel="noopener"` + `target=_blank` on external links with "Opens in a new tab" in the accessible name, `:focus-visible` outlines, `prefers-reduced-motion` support, lazy-loaded map.
- **(fix)** Exactly **one H1** (hero); one H2 per section; H3 for list titles; no headings used purely for styling (team quote is a `<blockquote>/<p>` styled like a heading).
- Images: WebP/JPG with width/height, `loading="lazy"` below the fold, `fetchpriority="high"` on first hero card, real alt text.
- One minified CSS, one `defer` JS, no jQuery, FA self-hosted `font-display: swap`.
- Targets: Lighthouse mobile **Performance ≥ 80, Accessibility ≥ 95, SEO ≥ 95, CLS < 0.1**.
- Organization/LocalBusiness JSON-LD from Settings. Optional FAQPage JSON-LD. **Do NOT** output Review structured data for own testimonials (violates Google guidelines).
- Reduced-motion: all animation/transition off, `.as-rv` visible, `scroll-behavior:auto`, every JS animation/autoplay respects it.

---

## 8. Behaviour must-haves (quick reference)

Theme toggle persists, no flash · nav hides on scroll-down / shows on scroll-up / scroll-spy · mobile menu locks scroll · progress bar · reveal stagger · hero line reveal · hero deck (autoplay 3800ms + click) · floating cursors (one follows mouse) · ring cursor · magnetic buttons · marquee pause on hover · service hover card + deck · counters count once (1600ms) · team stack + arrows + hover socials + autoplay (7000ms) · testimonial FLIP (520ms) · FAQ +/− · map click-to-interact · FAB (hover / tap / keyboard / Esc / outside-click) · back-to-top · footer wordmark fits exactly · form validation + success.

---

## 9. Definition of done (per phase)

- [ ] Files created and listed.
- [ ] Respects every hard constraint in §2 (spot-check: no Pro widget, no global selector, no hard-coded copy).
- [ ] Section-independence respected (delete-one-section thought test passes).
- [ ] 0/1/2/N item edge cases handled for any collection touched.
- [ ] Dark **and** light themes handled.
- [ ] Reduced-motion handled.
- [ ] No `undefined`, no console errors (as far as verifiable).
- [ ] Deviations from original documented; unverified items flagged.
- [ ] `tracker.md` updated.

---

## 10. Things the AI must NOT do

- Don't invent features, colours, copy or settings not in these files.
- Don't use Elementor Pro features or paid plugins.
- Don't hard-code editable text/URLs/colours in PHP templates or JS.
- Don't port dead code (`design.md` §10).
- Don't ship broken Elementor JSON, nor claim untested behaviour was verified.
- Don't skip ahead past a phase boundary without the user's "continue".
- Don't rewrite or "improve" the design. The goal is fidelity (plus the listed **(fix)** items).
