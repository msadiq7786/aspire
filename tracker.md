# tracker.md — Progress, Decisions & Handoff

> **This is the only file that changes during the project.** Update it at the end of every phase/session.
> **Read order:** `rules.md` → `architecture.md` → `design.md` → `tracker.md`.

---

## ▶ CURRENT STATE (update first)

| Field | Value |
|---|---|
| **Current phase** | **Phase 1 — Foundation (not started)** |
| **Last completed** | Phase 0 — Analysis & documentation (this four-file set) |
| **Next action** | Start Phase 1: create the `aspire-core` plugin skeleton per `architecture.md` §3, then tokens/base CSS, FA self-host, `core.js` registry, theme toggle + head script, settings framework, roles. |
| **Blocked on** | `assets/` folder not yet provided (see Open Questions Q1). Phase 1 can start without it; Phase 3+ needs it. |
| **Last updated** | 2026-10-09 |
| **Last model/session** | Claude (analysis session) |

**Nothing has been built yet.** No plugin code, CSS, JS, or Elementor JSON exists. Everything below is unchecked unless noted.

---

## 1. Phase tracker

Legend: ☐ not started · ◐ in progress · ☑ done · ⚠ done with caveats

| # | Phase | Status | Notes |
|---|---|---|---|
| 0 | Analysis & docs (`rules/architecture/design/tracker.md`) | ☑ | Derived from `index.html` + `Aspire-HTML-to-WordPress-Elementor-Master-Prompt.md` |
| 1 | Foundation | ☐ | |
| 2 | Globals | ☐ | |
| 3 | Sections 1/2 (Hero, Marquee, Services, Why, Stats) | ☐ | |
| 4 | Sections 2/2 (Founder, Team, Testimonials, FAQ, Contact + CF7) | ☐ | |
| 5 | Content & import | ☐ | |
| 6 | QA / acceptance | ☐ | |

### Phase 1 — Foundation
- [ ] `aspire-core.php` + `includes/` classes + `templates/` stubs (structure per `architecture.md` §3)
- [ ] `class-assets.php` — enqueue pipeline; concatenate CSS in order → `aspire.min.css`; JS → `aspire.min.js` (defer, no jQuery)
- [ ] `tokens.css` (dark + light tokens, `--as-gap`) and `light-theme.css` (all 73 original overrides, scoped)
- [ ] `base.css` (scoped hygiene, focus-visible, smooth scroll, anchor offsets) and `components.css` (`.as-btn` skin + 4 tile colours + alt + `.as-lin` + `.as-link`, `.as-pill` variants, `.as-frame`, `.as-ph`, `.as-grid-bg`, `.as-rv` + `as-d1…d6`)
- [ ] Font Awesome 6.5.2 Free self-hosted (CSS + webfonts, `font-display: swap`)
- [ ] `core.js` module registry (`AS.register`, `data-as` roots, try/catch)
- [ ] Theme toggle module + **inline head script at priority 1** (no flash)
- [ ] Settings framework (tabs per `architecture.md` §7) with defaults + Reset
- [ ] `class-roles.php` — "Site Editor" role
- [ ] Reduced-motion + editor/no-JS visibility rules for `.as-rv`
- [ ] Phase report written (files, deviations, unverified)

### Phase 2 — Globals
- [ ] Header (pill nav, hide/show on scroll, stuck state, logo from Site Identity, Primary menu, CTA, theme toggle, burger)
- [ ] Scroll-spy from menu hrefs; auto-hide links to missing sections
- [ ] Mobile overlay menu (≤900px, scroll lock, Menu/Close, aria-expanded)
- [ ] Progress bar (admin-bar offset)
- [ ] Cursor ring (hover-capable, rAF paused when hidden/idle)
- [ ] Magnetic buttons
- [ ] Ambient background (`.as-bgfx`, blobs + noise file) with toggles
- [ ] FAB (hover/tap/focus-visible/Esc/outside-click; items from Settings + custom items)
- [ ] Footer (4 columns, menus *Footer Navigate* / *Footer Services*, wordmark fit, `{year}`, back-to-top)
- [ ] `[as_social]`, `[as_contact_strip]`, `[as_map]` (+ click-to-interact, dark filter)

### Phase 3 — Sections 1/2
- [ ] Hero: Elementor layout, `[as_hero_stack]`, H1 runtime line-wrap reveal, deck (3800/550ms, click, pause off-screen), floating cursors (3, #2 follows mouse), 0/1-card edge cases, `100svh`
- [ ] Marquee: `[as_marquee]`, clone-to-fit, constant px/sec, pause on hover, admin options
- [ ] Services: `[as_services]`, list↔deck, floating card >900px, inline paragraph ≤900, auto numbers, secondary image field
- [ ] Why: Elementor layout + glass chip, `[as_why]`, icon dropdown + custom class
- [ ] Stats: `[as_stats]`, clipped-corner cards, count-up 1600ms, real final number in HTML, empty URL → non-link

### Phase 4 — Sections 2/2
- [ ] Founder
- [ ] Team: `[as_team]`, stack + arrows, quote fade 260ms, autoplay 7000ms with pause/resume, per-person social pills, aria-live, 1/2/N members
- [ ] Testimonials: `[as_testimonials]`, FLIP 520ms, breakpoint-aware visible count, arrows hidden if ≤ visible, guards for 0/1/2, initials ignore empty words, optional swipe/autoplay
- [ ] FAQ: `[as_faq]` with `<details>`, sticky intro (`overflow: clip`), single-open option, optional FAQPage JSON-LD
- [ ] Contact: CF7 form template, floating labels (wrap-aware), validation parity, error-clear JS, success box, honeypot + time-trap, Flamingo, SMTP notice, service dropdown auto-fill, autop off

### Phase 5 — Content & import
- [ ] CPT admin UX: drag-drop ordering, Duplicate, thumbnail/key columns, helper text, Draft = hidden
- [ ] Demo importer (idempotent): Home page (Canvas, front page), all items, menus, settings, CF7 form, Elementor JSON
- [ ] Per-section "Restore this section" JSON + full-page JSON (valid, or programmatic `_elementor_data` writer)
- [ ] Elementor Site Settings applied (width 1240, widgets space 0, container padding/gap 0, global colours/fonts)
- [ ] Dashboard widget "How to edit your website"
- [ ] `ADMIN-GUIDE.md` (includes the "Where to change what" table), `readme.txt`
- [ ] Demo images converted (`sadiq.ico` → JPG/WebP etc.) and bundled in `assets/img/demo/`

### Phase 6 — QA
- [ ] Run §4 acceptance checklist; record results and fixes below
- [ ] Final deliverables: plugin ZIP structure, Elementor JSONs, ADMIN-GUIDE.md, deviation changelog

---

## 2. Section build status

| Section | ID | Elementor layout | Plugin/shortcode | JS module | Dark ✓ | Light ✓ | Edge cases ✓ | Notes |
|---|---|---|---|---|---|---|---|---|
| Nav | — | n/a | ☐ | ☐ | ☐ | ☐ | ☐ | |
| Hero | `top` | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | |
| Marquee | `marquee` | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | |
| Services | `services` | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | |
| Why | `why` | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | |
| Stats | `stats` | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | |
| Founder | `about` | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | |
| Team | `team` | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | |
| Testimonials | `testimonials` | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | |
| FAQ | `faq` | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | |
| Contact + Map | `contact` | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | |
| Footer | — | n/a | ☐ | ☐ | ☐ | ☐ | ☐ | |
| FAB | — | n/a | ☐ | ☐ | ☐ | ☐ | ☐ | |

---

## 3. Open questions & risks

| ID | Item | Status / Proposed handling |
|---|---|---|
| Q1 | **`assets/` folder was not provided** with the analysis upload. Needed: `logo.svg`, `hero-one…six.jpg`, `why-aspire.jpg`, `founder.jpg`, `sadiq.ico`, `ikram.png`, `meezan.png`, `google-ads.jpg`, and the four JS-only service images (`web-development.jpg`, `seo.jpg`, `brand-identity.jpg`, `media-marketing.jpg`). | **Open.** Ask user to supply before Phase 3. Until then use the original gradient/SVG fallbacks. |
| Q2 | **Brand-name inconsistency:** "Aspire Studios" (most places) vs "Aspire Digital Solutions" (map iframe title, one testimonial text, email domain `aspiredigitalsolutions.in`). | Handle via *Business name* + *Alternate name* settings. Seed copy exactly as in source; ask user whether to unify. |
| Q3 | All social links in source are `href="#"`. | Seed empty; empty = hidden (rule §25). Ask user for real URLs. |
| Q4 | "View all services" button targets `#services` (self-link). | Keep URL editable; ask user for intended destination. |
| Q5 | Master prompt says only Testimonials uses the `--surface-2` band; the HTML also applies it to **Team** (`.team-section`). | **Resolved:** treat both as `--surface-2` with `border-block` (HTML wins). |
| Q6 | Master prompt says "~60" light-theme overrides; actual count in source is **73** `data-theme="light"` rules. | **Resolved:** port all of them. |
| Q7 | Cannot confirm a live WordPress environment is available to test (Elementor JSON import, CF7, Flamingo). | **Open.** Whoever builds must state what was and wasn't runnable. |
| Q8 | Recipient email default `marketing@aspiredigitalsolutions.in` — confirm still correct. | Open (low risk; editable in Settings → Form). |

---

## 4. Acceptance checklist (Phase 6)

**Visual:** side-by-side with `index.html` at 1440 / 1024 / 900 / 768 / 390 px, dark **and** light — no visible difference (fonts, spacing, colours, shadows, clip-corners, frames, glass).
- [ ] 1440 dark · [ ] 1440 light · [ ] 1024 dark · [ ] 1024 light · [ ] 900 dark · [ ] 900 light · [ ] 768 dark · [ ] 768 light · [ ] 390 dark · [ ] 390 light

**Behaviour parity:**
- [ ] theme toggle persists, no flash · [ ] nav hide/show + scroll-spy · [ ] mobile menu scroll lock · [ ] progress bar · [ ] reveal stagger · [ ] hero line reveal · [ ] hero deck (autoplay + click) · [ ] floating cursors (one follows mouse) · [ ] ring cursor · [ ] magnetic buttons · [ ] marquee pause on hover · [ ] service hover card + deck · [ ] counters count once · [ ] team stack/arrows/hover socials/autoplay · [ ] testimonial FLIP · [ ] FAQ +/− · [ ] map click-to-interact · [ ] FAB (hover, tap, keyboard, Esc, outside click) · [ ] back-to-top · [ ] footer wordmark fits exactly · [ ] form validation + success

**Independence (console must stay clean; spacing balanced):**
- [ ] Delete each of the 10 sections one at a time (nav links vanish, others work, no scroll-spy errors)
- [ ] Delete footer; delete FAB (separately)
- [ ] Every collection at 0, 1, 2 items
- [ ] Stress counts: 8 services, 6 team, 12 testimonials, 10 FAQs, 14 marquee words, 8 hero cards, 6 stats, 6 why points — no `undefined`, no broken layout
- [ ] Each section "Hide on mobile only"

**Editing (as "Site Editor", no code):**
- [ ] change heading / paragraph / button label+link / image · [ ] change phone/email/address (appears everywhere) · [ ] change accent colour + font · [ ] reorder + add testimonial · [ ] hide FAQ via Draft · [ ] toggle an effect · [ ] restore deleted section from JSON

**Form:**
- [ ] valid submit → email received + Flamingo entry · [ ] each invalid case shows correct red message · [ ] honeypot blocks bots · [ ] works with caching on

**Quality:**
- [ ] no console errors (frontend + Elementor editor) · [ ] reduced-motion respected · [ ] keyboard-only navigation · [ ] Lighthouse mobile Perf ≥ 80 / A11y ≥ 95 / SEO ≥ 95 / CLS < 0.1 · [ ] passes with WP Rocket/LiteSpeed "delay JS" exclusions

---

## 5. Decision log

| Date | Decision | Reason |
|---|---|---|
| 2026-10-09 | Four-file knowledge pack created (rules / architecture / design / tracker) so any model can resume without re-scanning the source. | Model-switch continuity |
| (pre-set) | Elementor Free + Hello Elementor + one `aspire-core` plugin; CPTs + shortcodes for collections; CF7 + Flamingo for the form; FA 6.5.2 CSS self-hosted. | Master prompt §2 — final, do not revisit |
| (pre-set) | Layout/responsiveness stays in plugin CSS (900/1000/1100/800/600/560), not Elementor's 1024/767. | Pitfall 29 |
| (pre-set) | Header/footer/FAB are plugin-rendered (no Theme Builder in Free). | Pitfall 7 |

---

## 6. Deviation log (every departure from `index.html`)

*Intentional **(fix)** items are pre-approved in the spec; record only the actual implementation here, plus any unplanned deviation.*

| Phase | Deviation | Why | Visual impact |
|---|---|---|---|
| — | *(none yet)* | | |

Pre-approved **(fix)** list to implement and then log: theme applied in `<head>` · tokens replace hard-coded colours · delays via classes not inline · `.as-rv` safety nets · admin-bar offsets · ring rAF pause · symmetric half-gap padding · module registry · scroll-spy from real links · hero line-wrap at runtime · deck poses extend to N · deck pauses off-screen · marquee constant speed · services per-item images · team autoplay resumes · team quote not an H2 · testimonial breakpoint-aware + guards · stats real number in HTML · FAQ sticky via `overflow: clip` · `100svh` · `data-network` hovers · real `<img>` instead of `--img`.

---

## 7. QA results log

| Date | Test | Result | Fix / follow-up |
|---|---|---|---|
| — | *(none yet)* | | |

---

## 8. Session handoff log (append newest at top)

### Template
```
### YYYY-MM-DD — <model name> — Phase N
Done:        - …
Files:       - …
Deviations:  - …
Unverified:  - …
Next action: - …
```

### 2026-10-09 — Claude — Phase 0 (analysis)
- **Done:** Read `Aspire-HTML-to-WordPress-Elementor-Master-Prompt.md` (full) and `index.html` (markup + JS in full; CSS structure, breakpoints, keyframes, z-index scale, light-override count, key rules). Produced `rules.md`, `architecture.md`, `design.md`, `tracker.md`.
- **Source facts confirmed:** CSS 12–3043, markup 3046–3567, JS 3568–3802; breakpoints 1400/1100/1000/900/800/600/560; 9 keyframes (one dead: `slide2`); 73 light-theme overrides; `.wrap` 1240/28px; section padding 120px.
- **Deviations from prompt noted:** Team also uses `--surface-2` band (Q5); light overrides 73 not ~60 (Q6).
- **Unverified:** `assets/` folder contents (Q1); the exact text of the 8 testimonials / 5 FAQ answers / 4 stat descriptions is *not* duplicated in `design.md` — read from `index.html` lines 3741–3752, 3406–3450, 3264–3308 when seeding `demo/content.json`.
- **Next action:** Begin Phase 1 (see CURRENT STATE).
