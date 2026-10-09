# architecture.md — Aspire Studios → WordPress (Elementor Free)

> **Purpose:** The technical blueprint: stack, plugin structure, data model, shortcodes, JS module system, form pipeline, build order. Follow it as written; do not re-decide.
> **Project in one line:** Convert the single-file `index.html` ("Aspire Studios" dark/light agency site, ~3,000 lines CSS, ~250 lines markup, ~235 lines JS) into a WordPress site a **non-technical person can maintain**, with **identical design, animation and behaviour**.
> **Read order:** `rules.md` → `architecture.md` → `design.md` → `tracker.md`.

---

## 1. Stack decisions (final)

| Layer | Choice |
|---|---|
| Theme | **Hello Elementor** (parent). Child theme optional. |
| Page builder | **Elementor Free** with **Flexbox Containers**. Home page template = **Elementor Canvas**. |
| Custom code | **One plugin: `aspire-core`** (CPTs, shortcodes, settings, CSS, JS, header, footer, FAB, importer, roles, CF7 hooks). |
| Header / footer / FAB | Rendered by the plugin via `wp_body_open` / `wp_footer` (Elementor Free has no Theme Builder; Canvas pages need this). Menus via *Appearance → Menus*, text via *Aspire Studios → Settings*. |
| Repeating collections | **Non-public Custom Post Types** rendered by shortcodes inside an Elementor **Shortcode widget**. |
| One-off content | Native Elementor Free widgets (Heading, Text Editor, Image, Button, Icon, Container). |
| Form | **Contact Form 7 + Flamingo** (stores every submission). SMTP plugin for delivery. |
| Icons | **Font Awesome 6.5.2 Free, self-hosted CSS + webfonts** (not the JS build). |
| Allowed plugins | Hello Elementor, Elementor, Contact Form 7, Flamingo, Safe SVG (or equiv.), WP Mail SMTP / FluentSMTP. SEO: Yoast/Rank Math free (optional). **No** paid/Pro plugins (ACF Pro, Pods Pro…). |
| Versions | PHP ≥ 8.0, WP ≥ 6.5, text-domain `aspire-core`, prefix `as_` / `as-`. |

---

## 2. Source file map (`index.html`) — use this instead of re-scanning

| Block | Lines |
|---|---|
| `<head>` (Inter font link, FA JS build) | 4–11 |
| `<style>` (all CSS) | 12–3043 |
| `<body>` markup | 3046–3567 |
| `<script>` (all JS) | 3568–3802 |

**CSS regions:** tokens/reset 12–165 · progress 155 · buttons 166 · nav 167 · image placeholders 338 · hero 356 · stats 548 · marquee 711 · services 751 · why 950 · founder bg 1110 · founder 1141 · testimonial 1229 · team 1381 · contact 1597 · map 1757 · footer 1839 · FAB 2254 · reveal/reduced-motion 2451–2470 · **light theme fixes 2470–2868** · FAQ 2869+.

**Markup:** nav 3050 · hero `#top` 3075 · marquee 3109 · services 3113 · why 3152 · stats 3264 · founder `#about` 3309 · team 3361 · testimonials 3382 · faq 3406 · contact+map 3451 · footer 3492 · FAB 3543.

**JS regions:** theme 3571 · marquee 3596 · reveal 3600 · counters 3609 · nav/scroll-spy/progress 3615 · mobile menu 3626 · hero deck 3631 · floating cursors 3643 · cursor ring 3659 · magnetic 3666 · services 3672 · team 3704 · testimonials 3741 · form 3776 · map 3784 · FAB 3787 · footer wordmark 3799.

---

## 3. Plugin file structure (create exactly)

```
aspire-core/
  aspire-core.php
  includes/
    class-cpt.php            class-settings.php       class-shortcodes.php
    class-header-footer.php  class-cf7.php            class-importer.php
    class-roles.php          class-assets.php         helpers.php
  templates/  header.php  footer.php  fab.php
  assets/
    css/  tokens.css base.css components.css header.css hero.css marquee.css
          services.css why.css stats.css founder.css team.css testimonials.css
          faq.css contact.css footer.css fab.css effects.css light-theme.css
          → concatenated, in that order, into aspire.min.css
    js/   core.js  modules/*.js  → aspire.min.js
    fonts/fa/   (FA 6.5.2 Free CSS + webfonts)
    img/demo/   (all site images, converted)
  demo/  content.json  home.elementor.json  sections/*.json (one per section)
  ADMIN-GUIDE.md
  readme.txt
```

Security standards on every handler: nonce + capability check, sanitise on input, escape on output.

---

## 4. Page structure & section independence

### 4.1 Elementor containers (one top-level container per section, Full Width)

| Section | CSS ID | Classes | JS root (`data-as`) |
|---|---|---|---|
| Hero | `top` | `as-sec as-hero` | `hero` |
| Marquee | `marquee` | `as-sec as-marquee-sec` | `marquee` |
| Services | `services` | `as-sec as-services` | `services` |
| Why Us | `why` | `as-sec as-why` | `why` |
| Social stats | `stats` | `as-sec as-stats` | `stats` |
| Founder | `about` | `as-sec as-founder` | `founder` |
| Team | `team` | `as-sec as-team` | `team` |
| Testimonials | `testimonials` | `as-sec as-testimonials` | `testimonials` |
| FAQ | `faq` | `as-sec as-faq` | `faq` |
| Contact + map | `contact` | `as-sec as-contact` | `contact` |
| Nav / Footer / FAB | — (plugin-rendered) | — | `nav` / `footer` / `fab` |

### 4.2 Independence guarantees
1. No section depends on another's markup, CSS, ID or JS. No cross-section sibling selectors. No negative margins overlapping neighbours. Decorative pseudo-elements stay inside their section (`overflow: clip`).
2. Spacing: symmetric half-gap `padding-block` via `--as-gap` (see `design.md` §4).
3. Nav and footer links **auto-hide** if their target section is missing (checked at load). `#top` with no hero scrolls to 0.
4. Empty collection → shortcode renders nothing; container collapses; no errors.
5. Global design-system classes (`.as-pill`, `.as-btn`, `.as-frame`, `.as-rv`) are independent of any section.

---

## 5. JavaScript architecture (module registry)

**Problem in source:** one monolithic script — one missing element (`$('#mq')`, `$('#stack')`, `$('#form')`, `$('#top')`, `top2.onclick`) kills everything after it.

**Design:**
```
core.js
  AS.register(name, init)
  on DOMContentLoaded: for each [data-as="name"] root → run registered init(root)
Each module:
  try { if (!root) return; query ONLY inside root; never assume another module exists } catch → console.warn
Global modules (also null-guarded): theme, progress bar, nav scroll-spy, back-to-top,
  cursor ring, reveal observer, magnetic buttons
```

| Module | Root | Responsibilities |
|---|---|---|
| `theme` | `<html>` | `data-theme`, `localStorage` `aspire-theme` (try/catch), toggle aria/title; **inline `<head>` script at priority 1** applies saved theme pre-paint |
| `reveal` | document | One `IntersectionObserver` (threshold .15, rootMargin `0 0 -6% 0`); stagger via `as-d1…d6`; 3s failsafe; re-init on Elementor `frontend/element_ready/shortcode.default` |
| `nav` | `nav` | Hide on scroll-down >400px, `stuck` >30px, scroll-spy built from **actual menu hrefs** (active = last target above 40% viewport), mobile overlay (scroll lock, Menu/Close) |
| `progress` | bar | Scroll %, admin-bar offset |
| `ring` | body | Cursor ring; pause rAF when tab hidden/idle |
| `magnetic` | `.as-mag` | Pointer pull |
| `hero` | `hero` | Line-wrap H1, card deck (3800/550ms, click), floating cursors; pause off-screen/hidden |
| `marquee` | `marquee` | Clone-to-fit (≥2× viewport, even number of identical halves), duration from track width (constant px/sec), pause on hover |
| `services` | `services` | List↔deck sync, floating card (>900px), keyboard focus |
| `stats` | `stats` | Count-up 1600ms ease-out quartic, decimals/thousands, reduced-motion instant |
| `founder` / `why` | — | Mostly CSS (reveal only) |
| `team` | `team` | Stack poses, arrows, quote fade (260ms), autoplay 7000ms **pause on hover/focus, resume on leave**, hover socials, aria-live |
| `testimonials` | `testimonials` | FLIP (WAAPI, 520ms), visible count by breakpoint (recomputed on resize), hide arrows if items ≤ visible, optional touch swipe/autoplay |
| `faq` | `faq` | Optional single-open; JSON-LD from PHP |
| `contact` | `contact` | Error-clearing on input, phone check, map click-to-interact |
| `fab` | `fab` | Hover (desktop) / tap (touch) / `:focus-visible` open; outside click + Esc close; `aria-expanded` |
| `footer` | `footer` | Wordmark fit on `resize`, `load`, `document.fonts.ready`; back-to-top |

**Pose-table rule:** every stack (hero, services, team) reuses the **last pose** for depths beyond the table, with `opacity:0; pointer-events:none`. Never index past the array.

**Loading:** one minified CSS file, one `defer` JS file, **no jQuery**. Exclude `aspire.min.js` and the inline theme script from cache-plugin "delay JS".

---

## 6. Data model (Custom Post Types — all non-public, no front-end URLs, excluded from search/sitemaps)

| CPT (admin label) | Fields | Seeded | Shortcode |
|---|---|---|---|
| **Hero Cards** | image, order | 6 | `[as_hero_stack]` |
| **Marquee Items** | title, optional link, order | 6 | `[as_marquee]` |
| **Services** | title, description (excerpt), main image (featured), optional secondary image, order | 4 | `[as_services]` (also feeds CF7 dropdown) |
| **Why Points** | icon (dropdown ~40 FA + custom class), title, text, order | 4 | `[as_why]` |
| **Social Stats** | icon, brand colour/gradient, number, suffix, description, link URL, new-tab, order | 4 | `[as_stats]` |
| **Team Members** | name (title), role, quote, photo (featured), LinkedIn/Instagram/X URLs, order | 3 | `[as_team]` |
| **Testimonials** | quote (editor), name (title), role/company, rating 1–5, source icon (Google/Facebook/Clutch/none), optional avatar, order | 8 | `[as_testimonials]` |
| **FAQs** | question (title), answer (editor), open-by-default checkbox, order | 5 | `[as_faq]` |

**Every collection must have:** short labelled fields with helper text; **drag-and-drop ordering** (`menu_order`); Draft = hidden; **Duplicate** row action; thumbnail + key-field list columns.

**Other shortcodes:** `[as_social context="hero|founder|contact|footer"]` (hover colours via `data-network`) · `[as_contact_strip]` · `[as_map]` · `[as_social_inline]` (inside CF7 form).

**Auto-generated (never typed):** service numbers `01, 02…`; initials circles (first letters of first two non-empty words); curly quotes (CSS); footer year (`{year}`); stat `aria-label`.

---

## 7. Settings (Aspire Studios → Settings, tabbed)

| Tab | Contents |
|---|---|
| General & Contact | business name, alternate name, tagline, email, phone, address, map override |
| Social Links | repeater: network (YouTube, Instagram, LinkedIn, X, Facebook, WhatsApp, GitHub, TikTok, Custom), URL, show-in (Hero/Founder/Contact/Footer) |
| Appearance | default theme (Dark/Light/System), toggle on/off, Dark & Light colours, heading/body font + weights, `--as-gap` |
| Effects | reveal (on/off, distance, duration), blobs, noise, progress bar, cursor ring, magnetic, nav hide-on-scroll, hero autoplay interval, floating cursors (+ names/colours), marquee speed/direction/pause/separator/size, team autoplay (0=off), testimonial autoplay, counter duration, FAB |
| Header & Footer | CTA text/URL/icon/tile, taglines, copyright with `{year}`, bottom text, wordmark, back-to-top |
| Form | recipient, subject, messages, options (email field, consent) |
| Advanced | Additional CSS, head/footer scripts, export/import settings JSON, **Reset to defaults**, cache-plugin compatibility notice |

**Other admin pieces:** menu locations *Primary*, *Footer Navigate*, *Footer Services*; **Install demo content** (one-click, idempotent) + per-section **Restore this section**; **"Site Editor" role** (edit pages with Elementor, edit all Aspire collections, upload media, edit Settings; no plugins/themes/users); **Dashboard widget** "How to edit your website".

---

## 8. Contact form pipeline

```
CF7 form (importer-created, wpautop OFF)
  → fields: your-name, your-service (auto-filled from Services CPT via wpcf7_form_tag),
            your-phone, your-message
  → validation: name ≥2 · service required · phone chars [+()-space digits], 7–15 digits
                (server filter wpcf7_validate_tel* + client check) · message ≥10
  → spam: honeypot + time-trap via wpcf7_spam; optional reCAPTCHA v3 / Turnstile
  → mail to Settings recipient (default marketing@aspiredigitalsolutions.in),
    subject "New enquiry from {name}", body: name, service, phone, message, page URL, date
  → Flamingo stores every message
  → success: styled .wpcf7-response-output → "Thanks! We've got your message and will get back to you soon."
  → admin notice if no SMTP plugin active
```
Floating labels must use `:focus-within` / `:has(:not(:placeholder-shown))` because CF7 wraps controls in `<span class="wpcf7-form-control-wrap">`. JS clears errors on input/change (CF7 does not). Source form **never submitted** (comment says "connect to Formspree / WP form handler here").

---

## 9. Elementor configuration (applied by importer)

- Site Settings → Layout: content width **1240**, widget space **0**, container padding **0**, gap **0**.
- Global Colors/Fonts set to brand palette.
- Home = **Elementor Canvas**, title hidden, set as front page.
- Each top-level container: Layout = Full Width, ID + classes per §4.1.
- Each section also shipped as a separate importable JSON in `demo/sections/` so a deleted section can be restored.
- If valid Elementor JSON can't be guaranteed → programmatic `_elementor_data` writer. **Never ship broken JSON.**
- Elementor **Free** widgets only: Container, Heading, Text Editor, Image, Button, Icon, Icon Box, Icon List, Social Icons, Divider, Spacer, Shortcode, HTML (static markup only), Menu Anchor.

---

## 10. Build phases (stop after each; wait for "continue")

| # | Phase | Scope |
|---|---|---|
| 1 | Foundation | Plugin skeleton, assets pipeline, tokens + light theme, base/components CSS (button skin, pills, frames, reveal), FA self-host, `core.js` registry, theme toggle + head script, settings framework, roles |
| 2 | Globals | Header, scroll-spy, mobile menu, progress, cursor ring, magnetic, FAB, footer + wordmark, back-to-top, social shortcode, contact-strip, map |
| 3 | Sections 1/2 | Hero (line reveal, deck, floating cursors), Marquee, Services, Why, Stats |
| 4 | Sections 2/2 | Founder, Team, Testimonials, FAQ, Contact + CF7 hooks |
| 5 | Content & import | CPT admin UX (ordering, duplicate, columns), demo importer, Elementor JSON per section + full page, CF7 + Flamingo config, dashboard widget, ADMIN-GUIDE.md |
| 6 | QA | Run acceptance checklist (`tracker.md` §4), report results and fixes |

Each phase: list files created, explain deviations from the original, list anything unverified.

---

## 11. Final deliverables
`aspire-core` plugin ZIP structure · Elementor JSON files (per section + full page) · short `ADMIN-GUIDE.md` for the owner · changelog of every deviation from the original HTML.
