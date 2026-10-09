# Aspire Studios: HTML → WordPress (Elementor Free) Master Prompt

**How to use**
1. Open an AI coding agent that can read/write files (Claude Code, Cursor, etc.).
2. Give it `index.html` and the `assets/` folder. Note that four service images (`web-development.jpg`, `seo.jpg`, `brand-identity.jpg`, `media-marketing.jpg`) are referenced **only from JavaScript**, so they will not show up if you search the HTML for `<img>`. Make sure they are in `assets/`.
3. Paste everything between **PROMPT START** and **PROMPT END**.
4. The prompt tells the agent to work phase by phase and stop for your "continue" after each phase.

---

# PROMPT START

## 0. ROLE AND GOAL

You are a senior WordPress, Elementor and front-end engineer. Convert the attached single-file site `index.html` ("Aspire Studios", a dark/light one-page agency site) into a production WordPress website that a **non-technical person can maintain**, while keeping the **design, animations and behaviour identical**.

`index.html` is the source of truth: about 3,000 lines of CSS, about 250 lines of markup and about 235 lines of JavaScript. Every colour, timing, easing, breakpoint and piece of copy in it is authoritative. Where this prompt says **(fix)**, the original has a flaw you must correct while keeping the look unchanged.

Seed all demo content **exactly** as it appears in the file. Counts to reproduce:
- 6 marquee words
- 4 services
- 6 hero cards
- 4 "why" points
- 4 social stats
- 3 team members
- 8 testimonials
- 5 FAQs
- 6 nav links
- 3 FAB items
- 4 form fields

## 1. HARD CONSTRAINTS

1. **Elementor FREE only.** Do not use or require any Pro feature (list in §12). If something needs Pro, build it in the plugin instead.
2. No paid plugins. Allowed free plugins: Hello Elementor (theme), Elementor, Contact Form 7, Flamingo, Safe SVG (or equivalent), and an SMTP plugin (WP Mail SMTP or FluentSMTP). Do not use ACF Pro, Pods Pro or similar.
3. **Every piece of text, image, link, colour, font, icon and repeated item must be editable from wp-admin.** No hard-coded copy in JS or PHP templates (defaults are seeded as editable content).
4. **Every section is independent.** Deleting or hiding any one section (or any item inside it) must never break another section, the layout, spacing or JavaScript (see §3).
5. Non-technical maintainability comes first: plain-English labels, helper text under every field, no shortcode editing needed for day-to-day changes.
6. Keep the exact visual output in both dark and light themes at all breakpoints (1440, 1100, 1000, 900, 800, 600, 560 and 390 px).
7. Zero JavaScript console errors in every state, including inside the Elementor editor.

## 2. ARCHITECTURE (decide nothing else; follow this)

| Layer | Choice |
|---|---|
| Theme | **Hello Elementor** (parent). Child theme optional. |
| Page building | Home page built in **Elementor Free** using **Flexbox Containers** (enable *Elementor → Settings → Features → Flexbox Container* if the site is older). Page template: **Elementor Canvas**. |
| Custom code | **One plugin: `aspire-core`**. It contains CPTs, shortcodes, settings page, CSS, JS, header, footer, floating button, demo importer, roles and CF7 hooks. |
| Header, footer, floating contact button | Rendered by the plugin via `wp_body_open` and `wp_footer`, so they appear on Canvas pages. Text and menus are editable in *Aspire Studios → Settings* and *Appearance → Menus*. (Elementor Free has no Theme Builder.) |
| Repeating collections | **Custom Post Types** (non-public, admin UI only) rendered by shortcodes dropped into an Elementor **Shortcode widget**. |
| One-off text/images/buttons | Native Elementor Free widgets (Heading, Text Editor, Image, Button, Icon, Container) so the person edits visually. |
| Form | **Contact Form 7** + **Flamingo** (stores every submission in wp-admin). |
| Icons | **Font Awesome 6.5.2 Free, self-hosted CSS + webfonts** (not the JS version). |

### Plugin file structure (create exactly)

```
aspire-core/
  aspire-core.php
  includes/ class-cpt.php  class-settings.php  class-shortcodes.php
            class-header-footer.php  class-cf7.php  class-importer.php
            class-roles.php  class-assets.php  helpers.php
  templates/ header.php  footer.php  fab.php
  assets/css/ tokens.css base.css components.css header.css hero.css marquee.css
              services.css why.css stats.css founder.css team.css testimonials.css
              faq.css contact.css footer.css fab.css effects.css light-theme.css
              + aspire.min.css (concatenated in that order)
  assets/js/  core.js  modules/*.js  + aspire.min.js
  assets/fonts/fa/   (FA 6.5.2 Free css + webfonts)
  assets/img/demo/   (all site images, converted: see §11 pitfalls 14, 15)
  demo/ content.json  home.elementor.json  sections/*.json (one per section)
  ADMIN-GUIDE.md  readme.txt
```

Coding standards: PHP ≥ 8.0, WP ≥ 6.5, WordPress Coding Standards, text-domain `aspire-core`, prefix everything with `as_` / `as-`, nonces + capability checks on every save, sanitise on input, escape on output.

## 3. SECTION INDEPENDENCE RULES (non-negotiable)

1. **One top-level Elementor container per section**, with a fixed CSS ID (anchor) and a class. Mapping:

   | Section | CSS ID | Class |
   |---|---|---|
   | Hero | `top` | `as-sec as-hero` |
   | Marquee | `marquee` | `as-sec as-marquee-sec` |
   | Services | `services` | `as-sec as-services` |
   | Why Us | `why` | `as-sec as-why` |
   | Social stats | `stats` | `as-sec as-stats` |
   | Founder | `about` | `as-sec as-founder` |
   | Team | `team` | `as-sec as-team` |
   | Testimonials | `testimonials` | `as-sec as-testimonials` |
   | FAQ | `faq` | `as-sec as-faq` |
   | Contact + map | `contact` | `as-sec as-contact` |

   Header, footer and floating contact button are plugin-rendered globals.
2. **No section may depend on another's markup, CSS, ID or JS.** No sibling selectors across sections. No negative margins that overlap neighbours. Decorative pseudo-elements stay inside their own section (`overflow: clip`).
3. **Spacing must not be coupled (fix).** The original relies on neighbours: Why has `padding-top: 0`, Stats has `padding: 0`, Founder/Contact use `140px 0 120px`. Give every section symmetric `padding-block` equal to **half** of the original gap on each side, so the visual gap is preserved and removing a section never collapses or doubles spacing. Expose the gap as one CSS variable `--as-gap` (default 120px).
4. **JS is a module registry (fix).** The original is one script; if any element is missing, `$('#x')` returns null and everything after it dies. New design:
   - `core.js` defines `AS.register(name, init)` and, on `DOMContentLoaded`, finds roots by `data-as="module-name"`.
   - Each module runs inside `try/catch`, starts with `if (!root) return`, queries only inside its own root, and never assumes another module exists.
   - Global modules (theme, progress bar, nav scroll-spy, back-to-top, cursor ring, reveal observer, magnetic buttons) also null-guard everything.
5. **Nav links auto-hide when their target section does not exist** (checked at load), including footer links. No dead anchors. `#top` with no hero scrolls to 0.
6. **Collections degrade gracefully** at 0, 1, 2 and N items (see each section). Empty collection → the shortcode renders nothing and the Elementor container collapses without errors.
7. Keep the shared design-system layer (tokens, `.as-pill`, `.as-btn`, `.as-frame`, `.as-rv`) global and independent of any section.

## 4. GLOBAL DESIGN SYSTEM AND FEATURES

### 4.1 Design tokens (CSS variables; all admin-editable in *Settings → Appearance*, separately for Dark and Light)

Dark defaults from the file:
`--bg #0B0B0C`, `--surface #111113`, `--surface-2 #17171A`, `--text #F5F5F5`, `--muted #A1A1AA`, `--subtle #71717A`, `--accent #4F46FF`, `--border rgba(255,255,255,.10)`, `--page-bg #0E0E0F`, `--font Inter`, `--ease cubic-bezier(.2,.7,.2,1)`.

Light defaults: `--page-bg/--bg #F4F6FA`, `--surface #FFFFFF`, `--surface-2 #E9EDF4`, `--text #191C26`, `--muted #555D6D`, `--subtle #737B8A`, `--border rgba(25,28,38,.14)`.

**(fix) Convert hard-coded colours into tokens** so admins can restyle: card background `#17171A`, input background `#0B0B0C`, marquee background `#08080A` (light `#E9EDF4`), service card `#262626`, accent tints. Every `rgba(79,70,255,…)` becomes `color-mix(in srgb, var(--accent) N%, transparent)` (affects FAB glow and ping, `.ok` box, testimonial/FAQ badges, FAQ icon hover, button wipe). Keep the lighter accent `#8B85FF` (active service number) derived from the accent. Keep all ~60 light-theme overrides from the file, scoped as `:root[data-theme="light"] .as-…`.

### 4.2 Theme toggle (dark/light)
- `data-theme="dark|light"` on `<html>`; saved in `localStorage` key `aspire-theme`, wrapped in `try/catch`.
- Button in the header: sun icon shown in dark mode, moon icon in light mode (inline SVG, 20 px, stroke 1.8). `aria-label` and `title` update to "Switch to light/dark theme". Hover: lifts 2 px and turns accent.
- **(fix)** Apply the saved theme with a tiny inline script at the very top of `<head>` (priority 1) so there is no flash of the wrong theme. The original sets it at the end of body.
- Admin options: default theme (Dark / Light / Follow system), show or hide the toggle. The original always defaults to dark.

### 4.3 Fonts
- Admin chooses **Heading font** and **Body font** (Google Fonts dropdown plus "custom font name" field) and weights (default Inter 300–700). The plugin enqueues the chosen families with `display=swap` plus a preconnect, and offers a "self-host fonts" switch.
- All text uses `--font-body` / `--font-heading`, falling back to `system-ui, sans-serif`. Do not force a font on Elementor widgets with high-specificity rules, so a per-widget font choice in Elementor still wins.
- Keep the fluid `clamp()` sizes from the original (h1 `clamp(34px,4.5vw,68px)` in hero, h2 `clamp(34px,4.6vw,68px)`, etc.). Elementor Free cannot author `clamp()`, so keep them in plugin CSS.

### 4.4 Ambient background
- `.as-bgfx` fixed, `z-index:-1`, `pointer-events:none`: three blurred (36px) rotated gradient ellipses drifting (`drift` 28s alternate, delays 0/-9s/-17s) plus an SVG feTurbulence noise overlay at `.07` opacity. In light theme the blobs are `.25` opacity. Render via `wp_body_open`. Admin toggles: blobs on/off, noise on/off.

### 4.5 Reveal on scroll
- Class `as-rv`: `opacity:0; translateY(34px)`; becomes `.in` with `transition: opacity .9s, transform .9s var(--ease)` and per-element delay.
- One `IntersectionObserver` (threshold `.15`, rootMargin `0 0 -6% 0`), unobserves after first reveal.
- **(fix) Stagger delays by classes** `as-d1…as-d6` (steps of `.08s` or `.1s`) and not by inline `style="--d:…"`, which WordPress strips for some roles and Elementor Free cannot set. Collections add delays automatically.
- Admin options: reveal on/off, distance, duration.
- **(fix) Safety:** content is visible when JS is off (`<noscript>` / `html:not(.as-js) .as-rv`), inside the Elementor editor (`body.elementor-editor-active`), and when a cache/"delay JS" plugin postpones the script (failsafe: reveal everything after 3s if the observer has not initialised). Do not mix with Elementor's own Entrance Animation on the same element.
- `prefers-reduced-motion: reduce`: all animation and transition off, `.as-rv` and hero text visible, `scroll-behavior:auto`; every JS animation and autoplay below also respects it.

### 4.6 Scroll progress bar
Fixed 2 px accent bar at the very top, width = scroll %, `z-index:100`. Toggle in settings. **(fix)** Offset by the WP admin bar when logged in.

### 4.7 Custom cursor ring
Only when `(hover:hover)` and not reduced-motion: 34 px white ring, `mix-blend-mode:difference`, follows the mouse with lerp `.18`; grows to 64 px and fills white over links, buttons, service list items, team cards, service cards and hero cards (excluding `.as-btn`, FAB items, stat cards, FAB main and back-to-top). The native cursor stays visible. Toggle in settings. **(fix)** Stop the `requestAnimationFrame` loop when the tab is hidden or the mouse is idle.

### 4.8 Magnetic buttons
On hover-capable devices, `.as-mag` buttons translate toward the pointer by `0.25×` (x) and `0.35×` (y) of the offset from centre, reset on leave. Toggle in settings.

### 4.9 Button system `.as-btn` (built on the Elementor **Button** widget, so text/link/icon are editable)
Style the widget's DOM (`.elementor-button`, `.elementor-button-text`, `.elementor-button-icon`) with a CSS skin:
- Height 52 px (46 px in the nav, 48 px in the form), radius 14, background `#F5F5F5`, text `#0B0B0C`, weight 600, 15 px.
- A square **icon tile** (height − 12 px, radius 10) sits at the right. On hover: accent colour wipes in from the right (`clip-path: inset(0 0 0 100%) → inset(0)`, .6s), label slides right, tile slides left and spins 360°.
- Tile colour variants via classes: `as-tile-blue` (`#202B3D` / icon `#5B9BFF`, "Let's talk"), `as-tile-purple` (`#302345` / `#C084FC`, "Start a project"), `as-tile-green` (`#203A32` / `#55D6A3`, "View all services"), `as-tile-amber` (`#3D2D20` / `#FFB454`, "Send message"). Keep light-theme variants.
- `as-btn--alt`: transparent with inset 1 px border.
- LinkedIn button (`.as-lin`): blue, with a moving "shine" sweep (3.6s) and blue glow on hover.
- Text-link style `.as-link` ("Why Aspire →", "Start your project →", "Still have a question?"): underline grows from the left on hover and the arrow shifts 6 px.

### 4.10 Pills / badges `.as-pill` (Elementor Heading widget, tag `span`)
Rounded 12 px chip with a small coloured square before the text. Colour classes: `as-pill--blue` (`#BBE0FF`/`#1E9BFF`, Services), `--green` (`#C9F5D4`/`#22C55E`, Why), `--purple` (`#E4C9FF`/`#B66DFF`, Stats and Founder tag), `--pink` (`#FFC4F2`/`#FF4FD8`, Team), `--accent` (tinted accent, Testimonials and FAQs).

### 4.11 Blueprint frame `.as-frame`
Dashed guide lines that extend 40 px beyond a rounded box (20 px under 1100 px), with 14 px corner arcs, built from layered gradients on `::before` (keep both `mask-composite` syntaxes). Used on the Why image, Founder image, Stats grid and Map. Colour `rgba(255,255,255,.34)`; light-theme variant.

### 4.12 Image placeholder `.as-ph`
Rounded 18 px (cards 10–12 px), `overflow:hidden`, subtle radial highlight overlay (`mix-blend-mode:overlay`). **(fix)** Use real `<img>` with `object-fit:cover` from the Media Library (Elementor Image widget or collection featured image), not inline `--img:url()` styles. If no image is set, show the original gradient fallback.

### 4.13 Section decorative grid `.as-grid-bg`
The 26 px grid plus scattered filled squares, masked, from `.fds:before/:after`. Make it an opt-in utility class used by Founder, Contact and Footer, each self-contained.

### 4.14 Global CSS hygiene
Scope **everything** under `.as-` (see §11 pitfall 4). Smooth scroll on `html`; anchor offsets with `scroll-margin-top`; `:focus-visible` 2 px accent outline, 3 px offset; `-webkit-font-smoothing:antialiased`; `body{overflow-x:hidden}`.

## 5. HEADER / NAV (plugin-rendered, `data-as="nav"`)

- Fixed floating pill: `top:16px`, max-width 1120, height 64, radius 18, glass (`rgba(20,20,23,.6)` + `backdrop-filter: blur(18px)`, border, shadow). Gets class `stuck` (darker background) after 30 px of scroll.
- **Hides on scroll down** (after 400 px, `translateY(-140%)`), **reappears on scroll up**; never hides while the mobile menu is open.
- Left: logo image (from *Appearance → Customize → Site Identity*, falls back to the bundled logo; 23×20) + business name text; hover rotates the logo 10° and scales .95.
- Centre (absolutely centred): menu from **WP menu location "Primary"** (default: Home `#top`, Services `#services`, About `#about`, Testimonials `#testimonials`, FAQs `#faq`, Contact `#contact`). Hover/active: underline scales in from the left. Admin can add, remove or reorder links in *Appearance → Menus*. Links may be anchors (`#x`) or full page URLs.
- **Scroll-spy (fix):** build the list from the menu's own hrefs in DOM order (the original hard-codes ids and crashes if a section is missing). The active link is the last target whose top is above 40% of the viewport height. Default `#top`.
- Right: CTA button "Let's talk" (icon `fa-comment-dots`, tile blue; text, URL, icon, tile colour editable in Settings → Header), theme toggle, burger.
- **Mobile (≤ 900 px):** CTA and centre links hidden; theme toggle pushed right; "Menu" button visible. Opening gives a full-screen overlay (`100dvh`, background `var(--bg)`, links 44 px/600 weight, left aligned, padding 36 px), button text becomes "Close", `aria-expanded` toggles, body scroll is locked, clicking any link closes it.
- **(fix)** Offset `top` by the admin-bar height (32 px desktop, 46 px mobile) when logged in.

## 6. SECTION SPECIFICATIONS

For each section: **Elementor** (visual editing), **Plugin** (collection/shortcode), **Behaviour**, **Edge cases**.

### 6.1 HERO (`#top`, `data-as="hero"`)
**Elementor:** two-column Container (`1.1fr / .9fr`, gap 56; stacks at ≤ 900 px), min-height `100svh` (fallback `100vh`), padding `130px 0 80px`. Widgets: eyebrow Text ("Aspire creative studio", 12.5 px, uppercase, `.14em` tracking), **Heading (H1, the only H1 on the page)**, Text Editor (lead), two Buttons (primary "Start a project" purple tile with rocket icon and `as-mag`; link "Why Aspire →" to `#why`), "Scroll down" link (circle with bobbing arrow, → `#services`), contact strip (email with envelope, phone with phone icon, "Location ↗" opening Google Maps in a new tab, each separated by a top border line).
**Plugin:**
- `[as_social context="hero"]`: social icons from Settings; hover colours by **`data-network`** (YouTube `#FF4040`, Instagram `#F56EA8`, LinkedIn `#3B9BFF`).
- `[as_hero_stack]`: the card deck, from CPT **Hero Cards** (image, order). 6 defaults. Container `500/560` aspect ratio, max 500.
- `[as_contact_strip]` (email/phone/location from Settings → General) so contact details are edited once.
**Behaviour:**
- **Headline mask reveal:** each line sits in an overflow-hidden `.hl` and slides up `translateY(105%)→0`, 1s, second line delayed `.12s`. **(fix)** Admin types one normal heading with line breaks; JS wraps each line at runtime (add `as-split-ready` before animating so it is fully visible without JS).
- **Card deck:** cards at `left 8% / top 8% / width 72% / height 84%`, radius 10. Pose table (translateX px, translateY px, rotate deg) for depth 0…5: `[0,0,-2] [44,-16,6] [-30,18,-7] [22,34,3] [-48,-14,-11] [60,22,10]`, `z-index = N−depth`, `brightness(1 − depth×.1)`. Every **3800 ms** (and on **click** anywhere on the stack) the front card flies out (`translate(115%,-6%) rotate(14deg)`), 550 ms later it becomes the back card. Lock while animating. Entrance `rise` animation (fade + 60 px). Autoplay off under reduced-motion. **(fix)** Pause when the hero is off-screen or the tab is hidden. **(fix)** Pose table must support any N: depths beyond 5 reuse the last pose with `opacity:0; pointer-events:none`.
- **"Figma-style" floating cursors:** 3 labelled cursors (arrow SVG plus name pill; originally *Meezan* `#4F46FF`, *Ikram* `#F24E8B`, *Sadiq* `#0FA958`) wander inside the hero (pick new random target every 1.8–4.0 s, lerp `.025`); the second one **follows the real mouse** with offset (+46, +34) and lerp `.09` while the pointer is inside the hero. **Admin:** on/off, names and colours (default: first three Team Members' first names, colours from a palette). Disabled under reduced-motion.
- Admin bar offset and `100svh` as above.
**Edge cases:** 0 cards → right column empties and the left column goes full width; 1 card → no shuffling.

### 6.2 MARQUEE (`#marquee`, `data-as="marquee"`)
**Plugin:** `[as_marquee]` from CPT **Marquee Items** (title, optional link, order). 6 defaults: Web development, SEO, Branding, Digital marketing, UI/UX, Content.
**Look:** full-width band, background `#08080A` (light `#E9EDF4`), border top and bottom, padding 26 px 0, text `clamp(22px,2.6vw,34px)` weight 500, colour `#D4D4D8`, 28 px side padding, accent "✦" separator after each item (symbol editable). `aria-hidden="true"`.
**Behaviour:** infinite leftward loop (`translateX(0 → -50%)`), **pauses on hover**. **(fix)** Original: 38 s fixed, list repeated 4× in JS. New: PHP renders items once (plus a no-JS duplicate); JS clones the set until the track is ≥ 2× the viewport width and always an even number of identical halves (seamless); duration is computed from the track width so **speed stays constant however many items are added**. Admin: speed (px/sec), direction (left/right), pause on hover, separator symbol, font size. Under reduced-motion: static, wrapped and centred.

### 6.3 SERVICES (`#services`, `data-as="services"`)
**Elementor:** pill "Our services" (blue), H2 "We build your / digital business.", lead paragraph, button "View all services" (`as-btn--alt`, green tile with layer-group icon, URL editable).
**Plugin:** `[as_services]` from CPT **Services**: Title, Description (excerpt), Main image (featured image), optional Secondary image (the small bordered overlay, bottom-right, 36% width, 4 px white border, shown originally only on SEO), order. Numbers `01, 02…` are **automatic**. 4 defaults.
**Layout:** two columns (gap 80): left = list (h3 `clamp(22px,2.5vw,36px)`, weight 500, muted; active item is white with a 2 px underline that grows, and the number in `#8B85FF`); right = stacked service cards (`min-height:540`, 360 on mobile).
**Behaviour:**
- Hover, keyboard focus or click on a list item activates it (list items are `tabindex=0`). Hovering on desktop (> 900 px) also shows the **floating description card** (dark `#262626`, 340 px wide, radius 6) positioned to the middle of the hovered row, clamped inside the section, sliding vertically between rows; it hides 180 ms after leaving unless the pointer enters the card. On ≤ 900 px the card is hidden and the active item's paragraph is shown inline.
- Image deck: cards stacked with pose table `[0,0,-2] [34,-10,5] [-30,14,-6] [54,-26,9]`, `brightness(1 − d×.16)`, transition `.8s`. Clicking a card activates that service.
- **(fix)** The original keeps per-service illustrations in a JS array (`art[]`) and image names in `serviceImages[]`, and shows the secondary image for `i===1` only. Replace with per-item fields and a generic fallback illustration. Poses beyond the table reuse the last pose, hidden. Service titles are `white-space:nowrap` on desktop; add helper text "keep titles under ~30 characters".
**Also feeds** the form's service dropdown (§7).
**Edge cases:** 1 service → no card deck motion; 0 → section collapses.

### 6.4 WHY US (`#why`, `data-as="why"`)
**Elementor:** two columns (`.9fr / 1.1fr`, gap 80, stacks at ≤ 900). Left: Container with `as-frame` containing an Image (4:5; 4:4.2 on mobile) and the **glass chip** at bottom-left (big "1:1" + "Work directly with the founder"; both editable, blur 14). Right: green pill "Why Aspire Studios", H2, lead, `[as_why]`, link "Start your project →".
**Plugin:** `[as_why]` from CPT **Why Points** (icon, title, text, order): 46 px circular icon (accent) + bold title + muted text; dashed top borders and a dashed bottom border on the last item; staggered reveal. 4 defaults (Founder-led, Built to convert, Fast and search-ready, Clear communication).
**Icon field:** dropdown of ~40 common Font Awesome icons plus a "custom class" box.

### 6.5 SOCIAL STATS (`#stats`, `data-as="stats"`)
**Elementor:** purple pill "Social stats", H2 "A community that / grows with us.", lead on the right (flex row, wraps).
**Plugin:** `[as_stats]` from CPT **Social Stats** (icon, brand colour or gradient, number, suffix, description, link URL, open-in-new-tab, order). 4 defaults: YouTube 20K+, Instagram 30K+ (gradient `#F9A03F→#E1306C→#833AB4`), LinkedIn 5K+, X 3K+.
**Look:** grid of 4 inside `as-frame` (→ 2 columns ≤ 1000, 1 column ≤ 560). Each card is a link: `#17171A`, 1 px border, **clipped top-right corner** (46 px cut) with a folded-corner triangle, faint blueprint grid pattern in the top-right (masked), 54 px white icon tile (`box-shadow 0 0 0 6px`). On hover/focus: lifts 4 px, lighter background, the icon tile takes the brand colour (`--b`) and tilts −8°, and an arrow fades in at the bottom-right rotated −45°.
**Behaviour:** number counts from 0 to target over **1600 ms** (ease-out quartic) when the card is revealed; `prefers-reduced-motion` shows the final value instantly. **(fix)** Output the real final number in the HTML (original shows "0" without JS and for SEO); support decimals and thousands separators; build the `aria-label` automatically ("YouTube: 20K+ subscribers. Opens in a new tab"). Placeholders `href="#"` are replaced by real URLs; cards with an empty URL render as non-links.

### 6.6 FOUNDER (`#about`, `data-as="founder"`)
**Elementor:** `.as-grid-bg` section, two columns (`.85fr / 1.15fr`, gap 80). Left: `as-frame` + square Image (fallback is a gradient). Right: lilac "Founder" tag, H2 "The Founder's Desk", two Text Editor paragraphs (name in bold, brighter than body; "Mohammed Rehan"), LinkedIn button (`as-lin`, new tab) and `[as_social context="founder"]`.
**Padding:** `140px 0 120px` expressed through §3 rule 3.

### 6.7 TEAM (`#team`, `data-as="team"`)
**Elementor:** pink pill "Team Members" and large quote icon (fa-quote-right, in a circle).
**Plugin:** `[as_team]` from CPT **Team Members**: Name (title), Role, **Quote**, Photo (featured image), LinkedIn, Instagram, X URLs, order. 3 defaults: Mohammed Sadiq (UI/UX Designer), Mohammed Ikram (WordPress Developer), Mohammed Meezan (SEO Expert), with the original quotes.
**Look:** left = quote as large heading (`clamp(28px,3.4vw,50px)`), short rule, bold name, small role. Right = stacked portrait cards (72% × 86%, radius 12, shadow) with round 48 px prev/next arrows overlapping the edges (±24 px; ±6 px on ≤ 900).
**Behaviour:**
- Pose table `[0,0,-2] [38,-12,5] [-30,14,-6]`, `brightness(1 − d×.2)`, front card gets `.front`.
- Changing member fades the quote out (260 ms swap, `translateY(12px)`) and in. Click a back card to bring it forward; arrows cycle; autoplay every **7000 ms** (admin: seconds, 0 = off).
- Hovering the **front** card reveals a floating pill of social icons (LinkedIn/Instagram/X) sliding up; **only icons with a URL are shown, each person has their own links** (the original shares one placeholder set).
- **(fix)** The original stops autoplay **forever** after the first hover. Pause on hover/focus and **resume on leave**. Pause when off-screen. Add `aria-live="polite"` to the quote and make cards keyboard-focusable buttons.
- **(fix)** The quote is semantically a `<blockquote>`/`<p>` styled like a heading (the original uses an H2, which pollutes the outline).
**Edge cases:** members > 3 → extra cards hidden behind; 2 members → two poses; 1 member → no arrows.

### 6.8 TESTIMONIALS (`#testimonials`, `data-as="testimonials"`)
**Elementor:** background `var(--surface-2)` with top/bottom borders; accent pill "Testimonials", H2 "People love us, you know.", lead.
**Plugin:** `[as_testimonials]` from CPT **Testimonials**: Quote (editor), Name (title), Role/Company, Rating 1–5, Source icon (Google / Facebook / Clutch / none), optional avatar, order. 8 defaults with the original text.
**Card look:** gold stars (`#F4B400`, `aria-label="5 out of 5 stars"`), grey Google brand icon at the top-right, quote, author row with an **auto-generated initials circle** (first letters of the first two words) + bold name + role. Min height 272, radius 8, padding 24/22.
**Layout:** 3 cards visible (gap 12) ≥ 901 px, 2 on ≤ 900, 1 on ≤ 600. Round arrows (48 px, scale 1.08 on hover) sit in the header.
**Behaviour:** arrows rotate the DOM order (next = first card moves to the end; prev = last moves to the front) and animate with a **FLIP** transition (Web Animations API, 520 ms, `var(--ease)`). Curly quotes `“ ”` are added by CSS (admin never types them).
**(fix):**
- The visible count must follow the breakpoint and be recomputed on resize.
- Hide the arrows when there are no more testimonials than visible cards.
- Guard 0/1/2 items (the original appends `undefined` with an empty list).
- Initials must ignore empty words.
- Optional: swipe/drag on touch and optional autoplay (admin seconds, 0 = off).
- Do **not** output Review structured data for your own testimonials (against Google's guidelines).

### 6.9 FAQ (`#faq`, `data-as="faq"`)
**Elementor:** two columns (`.82fr / 1.18fr`, gap `clamp(48px,9vw,132px)`; stacks at ≤ 800). Left, **sticky** at 120 px: accent pill "FAQs", H2 "Good to know before we begin." (max 9ch), lead, link "Still have a question? Get in touch →".
**Plugin:** `[as_faq]` from CPT **FAQs** (question = title, answer = editor, "open by default" checkbox, order). 5 defaults. Markup: native `<details class="as-faq-item"><summary>`; the +/− icon (34 px circle; the vertical bar rotates away when open; border/background turn accent on hover/open). Keep original paddings and responsive tweaks.
**Options:** "only one open at a time" (default off, as in the original). Optional FAQPage JSON-LD from the items.
**(fix)** Sticky only works if no ancestor has `overflow:hidden/auto`: use `overflow: clip` and keep Elementor containers `overflow: visible` here.

### 6.10 CONTACT + MAP (`#contact`, `data-as="contact"`)
**Elementor:** `.as-grid-bg` section; max-width 820 centred; pill "Get in touch", H2 "Let's start something / worth building.", a Shortcode widget with `[contact-form-7 id="…"]`, then `[as_map]`.
**Form (see §7):** Name, Service (dropdown), Phone, Message, "Send message" button (amber tile with paper-plane), social icons to its right (`[as_social context="contact"]`), success box.
**Map `[as_map]`:** Google Maps iframe built from the address in Settings (embed by query, no API key), inside `as-frame`; height `clamp(260px,34vw,380px)`, radius 12; **dark-mode filter** `invert(92%) hue-rotate(180deg) saturate(.75) contrast(.95)` (none in light theme); `loading="lazy"`; `referrerpolicy="no-referrer-when-downgrade"`; glass "Open in Google Maps" chip bottom-left. **Interaction:** the iframe ignores the pointer (so page scroll isn't trapped) until the map is clicked; it releases on mouse leave or a click outside. Admin may override with a full embed URL, and may disable the map.

### 6.11 FOOTER (plugin-rendered, `data-as="footer"`)
- `.as-grid-bg`. Four columns: **Brand** (logo + name, tagline "We build digital experiences for ambitious businesses.", `[as_social context="footer"]` Instagram/LinkedIn/YouTube/X), **Navigate** (WP menu location "Footer Navigate": Home, About, Team, FAQs, Contact), **Services** (menu "Footer Services": Web, SEO, Branding, Marketing), **Contact** (email, phone, address with external-link icon, from Settings).
- **Giant wordmark** (business name) auto-scaled to exactly fill the container width: set 100 px, measure, scale; re-run on `resize`, `load` and `document.fonts.ready`; `aria-hidden`.
- Bottom bar: "© {year} Aspire Studios. All rights reserved." (**year automatic**), "Made with care in India" (editable), round back-to-top button (smooth scroll to top) with 70 px right margin under 1400 px so it clears the floating button.

### 6.12 FLOATING CONTACT BUTTON ("expand button", plugin-rendered, `data-as="fab"`)
- Fixed bottom-right (22 px), `z-index:10000`. Main button: 60 px, radius 14, accent with glow and a continuous **ping ring** (`fping` 2.6s), `fa-comment-dots`.
- Opens a vertical fan of 3 items (top to bottom: **Location** "Location ↗" to Google Maps, **Phone** number, **Email** address), each = glass label pill + 52 px round icon; items fade/scale in, staggered; hovering an item inverts the label and tilts/accents the icon. Main button turns white when open.
- **Desktop:** opens on hover, closes on leave. **Touch:** tap toggles. **Keyboard:** opens on `:focus-visible`, closes when focus leaves. **Outside click and Esc** close it. `aria-expanded` and `aria-controls` maintained.
- **Items come automatically from Settings → General (phone/email/address); admin can add** WhatsApp (`wa.me`) or any custom item (label, URL, icon) and reorder. Admin toggle to hide the button. Light-theme styles retained.

## 7. CONTACT FORM (Contact Form 7 + Flamingo)

Elementor Free has no Form widget, so use CF7.

**Form template** (importer creates it; also fixes the CF7 gotchas in §11 pitfall 6):
```html
<div class="as-f">[text* your-name id:as-name minlength:2 autocomplete:name placeholder " "]<label for="as-name">Name</label><small>Enter your name.</small></div>
<div class="as-f as-sel">[select* your-service id:as-service first_as_label "Select a service"]<label for="as-service">Service</label><small>Choose the service you need.</small></div>
<div class="as-f">[tel* your-phone id:as-phone autocomplete:tel placeholder " "]<label for="as-phone">Phone number</label><small>Enter a valid phone number.</small></div>
<div class="as-f">[textarea* your-message id:as-msg minlength:10 rows:5 placeholder " "]<label for="as-msg">Tell us about your project</label><small>Add a few details about your project.</small></div>
<div class="as-frow"><button type="submit" class="as-btn as-tile-amber as-mag">Send message</button>[as_social_inline]</div>
```
**Requirements**
1. **Floating labels** (label floats up on focus, filled, or select) must work with CF7's `<span class="wpcf7-form-control-wrap">` wrapper: use `:focus-within`, `:has(:not(:placeholder-shown))` and the wrap-adjacent selector. The original relies on `input + label` siblings, which CF7 breaks.
2. **Validation parity:** name ≥ 2 chars; service required; phone allows only `+ ( ) - space digits` and **7–15 digits** (server-side filter on `wpcf7_validate_tel*`, plus instant client-side check); message ≥ 10 chars. Error state = red border `#F87171` and the field's own `<small>` message in red; errors clear as the user types/changes (CF7 doesn't do this, so add JS). Hide CF7's default tip text; keep the original messages.
3. **Service dropdown is automatic:** a `wpcf7_form_tag` filter fills the options from published **Services** (ordered). Adding a service adds the option.
4. **Success state:** style `.wpcf7-response-output` like the original `.ok` box (accent border, tinted background, 14 px radius), message "Thanks! We've got your message and will get back to you soon.", form resets. Hide the CF7 spinner or restyle it; disable the button while sending.
5. **Disable CF7 `wpautop`** for this form (`wpcf7_autop_or_not` → false) so no stray `<p>/<br>` break the layout.
6. **Delivery:** mail to the address in Settings (default marketing@aspiredigitalsolutions.in), subject "New enquiry from {name}", body with name, service, phone, message, page URL, date. Optional auto-reply. **Flamingo stores every message** (admin menu "Flamingo → Inbound Messages"). Add a built-in **honeypot + time-trap** via the `wpcf7_spam` filter; optional reCAPTCHA v3/Turnstile keys in Settings. Show an admin notice if no SMTP plugin is active.
7. Optional (settings toggles): email field, consent checkbox.

## 8. ADMIN EXPERIENCE (what the non-technical person sees)

**Left-menu group "Aspire Studios"** containing:
- **Settings** (tabs): *General & Contact* (business name, alternate name, tagline, email, phone, address, map override) · *Social Links* (repeater: network [YouTube, Instagram, LinkedIn, X, Facebook, WhatsApp, GitHub, TikTok, Custom], URL, show in: Hero / Founder / Contact / Footer) · *Appearance* (default theme, toggle on/off, colours for Dark and Light, heading font, body font, weights, section gap) · *Effects* (every toggle and number in §4, §6: reveal, blobs, noise, progress bar, cursor ring, magnetic, nav hide-on-scroll, hero autoplay interval, floating cursors + names, marquee speed/direction, team autoplay, testimonial autoplay, counter duration, FAB) · *Header & Footer* (CTA, taglines, copyright with `{year}`, bottom text, wordmark, back-to-top) · *Form* (recipient, subject, messages, options) · *Advanced* (Additional CSS since Elementor Free has no custom CSS, head/footer scripts for analytics, export/import settings JSON, **Reset to defaults**).
- **Collections** as submenus: Hero Cards, Marquee Items, Services, Why Points, Social Stats, Team Members, Testimonials, FAQs.
- **Install demo content** (one click, idempotent): creates the Home page (Elementor Canvas, set as front page), all collection items, menus, settings, the CF7 form, and imports the Elementor JSON. A per-section **"Restore this section"** import is also available.

**Every collection must have:**
- A clean edit screen: short labelled fields with helper text (e.g. "Photo: square image, at least 800 px"), no developer jargon.
- **Drag-and-drop ordering** in the list table (saves `menu_order`).
- **Draft = hidden** (explain in helper text: "Set to Draft to hide this without deleting").
- A **Duplicate** row action, thumbnail and key-field columns.
- Items non-public (no front-end single URLs, excluded from search and sitemaps).

**Roles:** create a **"Site Editor"** role (edit pages with Elementor, edit all Aspire collections, upload media, edit Settings; no plugins/themes/users). Add a **Dashboard widget "How to edit your website"** with the 6 most common tasks and links.

**Where to change what** (put this table in ADMIN-GUIDE.md and the dashboard widget):

| I want to change… | Go to |
|---|---|
| Any heading/paragraph/image/button text in a section | Pages → Home → Edit with Elementor → click the text |
| Phone, email, address, social links | Aspire Studios → Settings |
| Add/remove/reorder marquee words, services, team, testimonials, FAQs, stats, why-points, hero cards | Aspire Studios → that collection |
| Menu links | Appearance → Menus |
| Colours, fonts, dark/light default | Aspire Studios → Settings → Appearance |
| Turn an animation or the cursor effect on/off | Aspire Studios → Settings → Effects |
| Hide a whole section | Elementor → select section → Advanced → Responsive → Hide, or delete it |
| See form enquiries | Flamingo → Inbound Messages |
| Logo | Appearance → Customize → Site Identity |

## 9. ELEMENTOR SETUP (importer applies; document it too)

- Site Settings → **Layout**: content width 1240; **widgets space 0**; container padding 0 and gap 0 (prevents the stray 10 px/20 px Elementor defaults that shift the design).
- Site Settings → **Global Colors/Fonts**: set to the brand palette and the chosen font so pickers show brand colours.
- Home page template **Elementor Canvas**; hide the page title.
- Every top-level container: Layout = Full Width, CSS ID and classes per §3.
- Use Elementor's free **Hide on Desktop/Tablet/Mobile** and **Entrance Animation** only if the admin wants (default off, since `as-rv` handles reveal).
- Provide each section as a separate importable JSON (Templates → Saved Templates → Import) so a deleted section can be restored.
- If valid Elementor JSON cannot be guaranteed, fall back to building each section with the importer's programmatic `_elementor_data` writer; do **not** ship broken JSON.

## 10. ACCESSIBILITY, PERFORMANCE, SEO (keep and extend)

- Keep: `lang`, skip-friendly structure, `aria-label`s on icon-only buttons and social links, `aria-expanded` on burger/FAB, `aria-hidden` on decorative items, `tabindex=0` on service rows, `rel="noopener"` + `target=_blank` on external links with "Opens in a new tab" in the accessible name, focus-visible outlines, reduced-motion support, lazy-loaded map.
- **(fix)** One H1; one H2 per section; H3 for list titles; no headings used purely for styling.
- Images: WebP/JPG, correct `width/height`, `loading="lazy"` below the fold, `fetchpriority="high"` on the first hero card; real alt text from the media library/name.
- Load CSS as one minified file, JS `defer` as one file; no jQuery dependency; FA CSS self-hosted with `font-display: swap`.
- Targets: Lighthouse mobile Performance ≥ 80, Accessibility ≥ 95, SEO ≥ 95, CLS < 0.1.
- SEO plugin (Yoast/Rank Math free) provides title/meta; add Organization/LocalBusiness JSON-LD from Settings.
- Compatible with caching/minify plugins (see pitfall 21).

## 11. KNOWN PITFALLS: YOU MUST IMPLEMENT THESE FIXES

| # | Problem found in this HTML / Elementor Free / WordPress | Why it breaks | Required solution |
|---|---|---|---|
| 1 | **One monolithic script.** `$('#mq')`, `$('#stack')`, `$('#form')`, `$('#top')`, `top2.onclick`, scroll-spy `$('#'+id).getBoundingClientRect()` all assume every element exists. | Deleting one section throws a TypeError and every later feature (nav, progress bar, carousels, FAB) dies. | Module registry with per-module `try/catch` and root guards (§3.4). Scroll-spy built from actual links. |
| 2 | **Arrays sized to today's content:** hero `hp` (6 poses), services `pos` (4), team `teamPositions` (3), `art[]`, `serviceImages[]`, `i===1` secondary image. | Adding a 5th service or 4th team member → `undefined` pose → crash or "undefined" text. | Pose tables extend by reusing the last pose hidden; per-item image fields; no index-based special cases. |
| 3 | **Content lives in JS:** marquee words, team, testimonials, service images, ghost-cursor names. | Not editable in WP, invisible to search engines, flashes empty on load. | CPTs rendered server-side in PHP; JS only enhances. |
| 4 | **Global selectors:** `nav`, `section`, `h1,h2,h3`, `a`, `button,input,textarea`, `*{margin:0;padding:0}`, and short classes/IDs (`.wrap .f .ok .err .in .on .hide .tag .row .k .sc .tc .tb`, `#n #s #p #m #bar #top #nav #ring`). | Collide with Elementor (`<section>` legacy, `<nav>` breadcrumbs/pagination), WP core and plugins; reset margins of Elementor widgets. | Prefix all classes `as-` and IDs/anchors via `data-as` (keep only the anchor IDs in §3); scope everything under `.as-sec`/`.as-nav`; no global element selectors; no universal reset outside `.as-`. |
| 5 | **Elementor defaults** (container padding 10 px, gap 20 px, widget bottom margin 20 px, kit typography on H1–H6, `line-height`, `letter-spacing`). | Shifts spacing and overrides headline tracking. | Reset in Site Settings (§9) and in plugin CSS with `.as-sec` specificity; keep clamp sizes in plugin CSS. |
| 6 | **No Form widget in Elementor Free; CF7 specifics.** CF7 wraps controls in `<span class="wpcf7-form-control-wrap">`, injects `<p>/<br>` (autop), shows its own tips, never clears errors on typing, strips whitespace placeholders, stores nothing. | Floating labels (`input + label`) stop working, layout breaks, no leads saved. | §7: wrap-aware CSS, autop off, `placeholder " "`, custom error `<small>`, JS to clear errors, Flamingo, honeypot, SMTP notice. |
| 7 | **No Theme Builder in Free** (header/footer). | Cannot build the nav/footer in Elementor. | Plugin renders header/footer/FAB with Settings + Menus. (Alternative if visual footer editing is wanted: free "Header, Footer & Blocks" plugin; keep behaviours via classes.) |
| 8 | **No Custom CSS, Custom Fonts, Motion Effects, Dynamic Tags, Loop Grid, Popup in Free.** | Cannot do carousels/collections/CSS in Elementor. | Plugin CSS/JS, CPTs + shortcodes, Settings → Advanced "Additional CSS", Google Fonts via Settings. |
| 9 | **`.as-rv` starts at `opacity:0`.** | Blank sections in the Elementor editor, with JS off, with "delay JS" caching, or if the observer fails. | Visible in editor/no-JS; 3s failsafe; re-init after Elementor re-renders a widget (`elementorFrontend.hooks.addAction('frontend/element_ready/shortcode.default', …)`); no duplicate with Elementor Entrance Animation. |
| 10 | **Theme set at end of `<body>`.** | Flash of dark theme for light-mode users. | Inline head script (§4.2). |
| 11 | **Font Awesome:** original uses the JS build with `autoReplaceSvg:"nest"`; Elementor's bundled FA lacks `fa-x-twitter`. | Late icon swap/layout shift; missing X icon. | Self-host FA 6.5.2 Free **CSS + webfonts**; do not load the JS build. |
| 12 | **`logo.svg`** blocked by WordPress uploads. | Upload fails or logo missing. | Safe SVG plugin (sanitises) or PNG/WebP fallback; Site Identity logo with bundled default. |
| 13 | **`sadiq.ico` used as a portrait.** | `.ico` can't be resized by WordPress, no srcset, poor quality. | Convert to JPG/WebP like the other team photos. |
| 14 | **Relative `assets/…` paths and `--img:url()` inline styles.** | Break after moving files; `--*` inline styles are stripped by WP's `style` filter and Elementor Free can't set them. | Media Library images with `<img>`/featured images; CSS references via plugin-relative URLs; seeded demo images copied into the Media Library by the importer. |
| 15 | **Admin bar overlap.** | Fixed nav and 2 px progress bar sit under the 32/46 px admin bar when logged in. | `body.admin-bar` offsets. |
| 16 | **`position:fixed` / `sticky` ancestors.** The mobile menu overlay is `fixed` inside the nav; FAQ intro is `sticky`. | Any ancestor with `transform/filter/backdrop-filter` re-anchors `fixed`; ancestors with `overflow:hidden` kill `sticky`. | Print nav/FAB directly under `<body>` via `wp_body_open`/`wp_footer`; use `overflow: clip` in FAQ ancestors. |
| 17 | **Inter-section padding is coupled** (Why `0` top, Stats `0`, Founder `140/120`). | Removing a section collapses or doubles spacing. | Symmetric half-gap padding per section (§3.3). |
| 18 | **Hero headline reveal needs per-line wrapper spans.** | A single Elementor Heading can't contain them. | Runtime line-wrapping (§6.1). |
| 19 | **Marquee loop** depends on the list being repeated 4× and a fixed 38 s. | Adding or removing words changes speed or leaves a gap. | Clone-to-fit + width-based duration (§6.2). |
| 20 | **Testimonial carousel** assumes 3 visible and ≥ 4 items; stars/Google icon hard-coded; initials crash on double spaces; no resize handling. | Wrong arrows/empty cards with few items; inaccurate review icons. | §6.8 fixes. |
| 21 | **Cache/optimisation plugins** (WP Rocket, LiteSpeed, Autoptimize, Perfmatters) "delay JS", combine CSS, inline small images. | Reveal never fires; data-URI noise/masks break; inline theme script delayed. | Exclude `aspire.min.js` and the inline theme script from delay/defer; keep the noise SVG as a file; provide a "Compatibility" notice in Settings listing exclusions; 3s reveal failsafe. |
| 22 | **WordPress KSES** removes `<svg>`, `<iframe>`, `<style>`, `<script>`, `style="--x:…"` from content for users without `unfiltered_html`. | Illustrations, map and inline styles vanish for Editor-role users. | Render SVG/iframe/scripts only from plugin code; use classes (not inline custom properties); don't put these in HTML widgets. |
| 23 | **Brand hover colours via `a[aria-label=YouTube]`.** | Breaks when the label is edited/translated. | `data-network="youtube"` attributes. |
| 24 | **Hard-coded accent RGBA / colours in ~10 places and SVG fills.** | Changing accent in settings leaves leftovers. | `color-mix()` tokens; SVG fills `currentColor`/CSS vars. |
| 25 | **Naming and placeholders:** "Aspire Studios" vs "Aspire Digital Solutions" (map title, review text, email domain); every social link is `#`; hero "View all services" → `#services` (self-link). | Inconsistent branding; dead links. | Business name in Settings (with alternate name field); placeholders empty = hidden; every URL editable. |
| 26 | **Dead/unused code in the HTML:** `.cred` strip + `slide2` keyframes, `.fd .row`, `.contact-fab.away/.in` (never toggled), `.fa-moon/.fa-sun` rules (icons are SVG `.ti-*`), `data-depth` on hero cards (never read), nav `.tb+.theme-toggle` rules declared 3×. | Bloat and confusion. | Do not port; consolidate once. Do not build admin options for them. |
| 27 | **Team autoplay stops forever after the first hover; infinite `requestAnimationFrame` loops** (ring, ghost cursors) and `setInterval`s run while off-screen. | Wasted CPU/battery; behaviour surprising. | Pause/resume with IntersectionObserver and `visibilitychange`. |
| 28 | **Hero uses `100vh`.** | Mobile browser toolbar jump. | `100svh` with `100vh` fallback. |
| 29 | **Responsive breakpoints differ from Elementor's (1024/767).** | Elementor responsive controls fight plugin CSS. | Layout/responsiveness stays in plugin CSS (900/1000/1100/800/600/560); Elementor is used for content, colour, images. |
| 30 | **Elementor Free container link / sticky / absolute positioning are not reliable in Free.** | Cards that need whole-card links or overlays can't be built visually. | Those pieces (stat cards, stacks, overlays) are plugin-rendered. |

## 12. ELEMENTOR FREE vs PRO: do not use these (Pro-only)

Form widget · Theme Builder (header/footer/archive/single) · Popup Builder · Posts / Loop Grid / Loop Carousel · Dynamic Tags and ACF integration · Custom CSS (per element and Site Settings) · Custom Fonts · Motion Effects (scroll/mouse) · Sticky · Nav Menu widget and Mega Menu · Slides / Testimonial Carousel / Price Table / Flip Box / Call To Action / Animated Headline / Countdown / Lottie · Global widgets · Role Manager · Custom Code. (Verify in the installed version; if a control is locked, use the plugin approach.)

**Free widgets to use:** Container, Heading, Text Editor, Image, Button, Icon, Icon Box, Icon List, Social Icons, Divider, Spacer, Shortcode, HTML (static markup only), Menu Anchor, Google Maps (not used; custom iframe for the dark filter).

## 13. WORK PLAN (stop after each phase and wait for "continue")

1. **Phase 1 (Foundation):** plugin skeleton, assets pipeline, tokens + light theme, base/components CSS (button skin, pills, frames, reveal), FA self-host, `core.js` registry, theme toggle with head script, settings framework, roles.
2. **Phase 2 (Globals):** header, scroll-spy, mobile menu, progress bar, cursor ring, magnetic, FAB, footer + wordmark, back-to-top, social shortcode, contact-strip, map.
3. **Phase 3 (Sections 1/2):** Hero (incl. line reveal, deck, floating cursors), Marquee, Services, Why, Stats.
4. **Phase 4 (Sections 2/2):** Founder, Team, Testimonials, FAQ, Contact + CF7 form hooks.
5. **Phase 5 (Content and import):** CPT admin UX (ordering, duplicate, columns), demo importer, Elementor JSON per section and full page, CF7 form + Flamingo config, dashboard widget, ADMIN-GUIDE.md.
6. **Phase 6 (QA):** run §14 and report results with fixes.

In every phase: show the files created, explain deviations from the original, and list anything you could not verify.

## 14. ACCEPTANCE CHECKLIST (all must pass)

**Visual:** side-by-side with `index.html` at 1440 / 1024 / 900 / 768 / 390 px in dark **and** light themes shows no visible difference (fonts, spacing, colours, shadows, clip-corners, frames, glass effects).

**Behaviour parity:** theme toggle persists with no flash · nav hides/shows on scroll and highlights the current section · mobile menu locks scroll · progress bar · reveal stagger · hero line reveal · hero deck (autoplay, click) · floating cursors (one follows the mouse) · ring cursor · magnetic buttons · marquee pauses on hover · service hover card + deck · counters count once · team stack, arrows, hover socials, autoplay · testimonial FLIP slide · FAQ +/− · map click-to-interact · FAB (hover, tap, keyboard, Esc, outside click) · back-to-top · footer wordmark fits exactly · form validation + success.

**Independence (run each separately, console must stay clean, spacing must stay balanced):**
- Delete each of the 10 sections one at a time: nav links to it vanish, other sections work, no scroll-spy errors.
- Delete the footer and the FAB separately.
- Set every collection to 0, 1 and 2 items; add: 8 services, 6 team members, 12 testimonials, 10 FAQs, 14 marquee words, 8 hero cards, 6 stats, 6 why points. No `undefined`, no broken layout.
- Set each section to Hide on mobile only.

**Editing (as a "Site Editor" user, no code):** change a heading, a paragraph, a button label/link, an image; change phone/email/address (appears everywhere); change accent colour and font; reorder and add a testimonial; hide a FAQ via Draft; toggle an effect; restore a deleted section from its JSON.

**Form:** valid submit → email received and Flamingo entry saved; each invalid case shows the correct red message; honeypot blocks bots; works with caching on.

**Quality:** no console errors (frontend and Elementor editor), reduced-motion respected, keyboard-only navigation works, Lighthouse targets met, passes with WP Rocket/LiteSpeed "delay JS" exclusions applied.

**Final output:** the `aspire-core` plugin ZIP structure, Elementor JSON files, a short ADMIN-GUIDE.md for the non-technical owner, and a changelog of every deviation from the original HTML.

# PROMPT END

---

## Reference: what the analysis found in `index.html` (for your own checking)

- **Order of sections:** Hero → Marquee → Services → Why → Stats → Founder (About) → Team → Testimonials → FAQ → Contact + Map → Footer, plus the floating contact button.
- **Data hidden in JS (must become editable content):** marquee words, team (3, with quotes), testimonials (8), the four service images, ghost-cursor names, hero/services/team pose tables.
- **Assets:** `logo.svg`, `hero-one…six.jpg`, `why-aspire.jpg`, `founder.jpg`, `sadiq.ico`, `ikram.png`, `meezan.png`, `google-ads.jpg`, and (via JS) `web-development.jpg`, `seo.jpg`, `brand-identity.jpg`, `media-marketing.jpg`.
- **Behaviour timings:** reveal `.9s`, hero headline `1s` (+`.12s`), hero deck `3800ms`/`550ms`, team `7000ms`/`260ms`, counters `1600ms`, testimonial FLIP `520ms`, service card hide `180ms`, marquee `38s`, nav hides after `400px`, stuck after `30px`, scroll-spy line at `40%` of viewport.
- **The form never submits in the original** (a code comment says "connect to Formspree / WP form handler here"), so the WordPress version is the first time submissions actually go anywhere.
